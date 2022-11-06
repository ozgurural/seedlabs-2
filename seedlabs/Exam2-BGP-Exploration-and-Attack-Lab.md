# SEEDlabs: Exam2 - BGP Exploration and Attack Lab

#### Ozgur Ural
#### Student ID: 2564455

### 2 The Lab Setup and the SEED Internet Emulator

First of all, I downloaded the emulator files. Then run the docker with the following commands.
```sh
$ docker-compose build
$ docker-compose up
```

### 3 Task 1: Stub Autonomous System
#### 3.1 Task 1.a: Understanding AS-155’s BGP Configuration

Task 1.a.1:  Find the peer autonomous system of AS-155, find the BGP route of 10.155.0.254/24, and use cat /etc/bird/bird.conf to query its configuration information.

![image](https://user-images.githubusercontent.com/4716254/200149098-ba95c405-c104-4bd7-a22c-dd4a2d569ab3.png)

It is known from the routing configuration that AS-150 is interconnected with 3 autonomous systems, of which p_as156 has a peer relationship with it:

![image](https://user-images.githubusercontent.com/4716254/200149125-4ae9cde5-716b-4730-9d89-98c65d072d89.png)


Task 1.a.2: AS-155 is interconnected with multiple ASs at the same time, one of which will not affect the AS-155's access to the Internet.

Here we choose to ping the host 10.156.0.72 from 10.155.0.72, and disconnect the links of the BGP routes one by one. If and only when all the links are disconnected, the ping command shows that it is unreachable.

![image](https://user-images.githubusercontent.com/4716254/200149151-9eee0fc3-2220-47fa-af75-121328c21040.png)


#### 3.2 Task 1.b: Observing BGP UPDATE Messages

Run the following command on the router to store the packets received by the router into a pcap file and transfer them to the virtual machine:

```sh
tcpdump -i any -w /tmp/pgp.pcap "tcp port 179"
```

Cut off a router connected to it (go offline or cut off all bgp connections), use wireshar to read the pcap file, find the UPDATE MESSAGE of BGP, and see the message that the route exits:

![image](https://user-images.githubusercontent.com/4716254/200150617-99dff9df-edd0-42f9-9de7-7ad988b55373.png)

Reconnecting can also capture routing update packets:

![image](https://user-images.githubusercontent.com/4716254/200150635-3e735faa-d2f1-4f95-a0e1-48adde93dd42.png)

#### 3.3 Task 1.c: Experimenting with Large Communities

First cut off the connection between AS-4 and AS-156, and then run the ping command on 10.156.0.71. It is found that 10.155.0.71 can be pinged, but 10.161.0.71 cannot be pinged. Although AS-156 is connected to the Internet through AS-155, due to the relationship between the two peers, AS-155 will not forward the data of AS-156 (whether it is forwarded depends on the relationship between the two).

![image](https://user-images.githubusercontent.com/4716254/200150667-5dc13072-70f9-4c19-90d5-ddce54a10875.png)

Modify the configuration file of the AS-155 router to realize that the data packets of the AS-156 are forwarded through the AS-155. There are two changes in total:

After completing the modification with the following command, 10.156.0.71 can ping 10.161.0.71:

```sh
$ dockps | grep 155
$ docker cp [docker id]:/etc/bird/bird.conf ./as155_bird.conf
$ docker cp ./as155_bird.conf [docker id]:/etc/bird/bird.conf
$ docker exec [docker id] birdc configure
```

#### 3.4 Task 1.d: Configuring AS-180

The experiments in this section need to configure a series of router configuration information to enable AS-180 to access the Internet. Use import bird conf.sh and export bird conf.sh to import and export configuration files in the container.

##### step 1 Connect AS-180 and AS-171
Add the following to the configuration of AS-180 and AS-171 respectively:

![1](./exam2/1.png)




### 4 Task 2: Transit Autonomous System
#### 4.1 Task 2.a: Experimenting with IBGP
#### 4.2 Task 2.b: Experimenting with IGP
#### 4.3 Task 2.c: Configuring AS-5
### 5 Task 3: Path Selection
### 6 Task 4: IP Anycast
### 7 Task 5: BGP Prefix Attack
#### 7.1 Task 5.a. Launching the Prefix Hijacking Attack from AS-161
#### 7.2 Task 5.b. Fighting Back from AS-154
#### 7.3 Task 5.c. Fixing the Problem at AS-3

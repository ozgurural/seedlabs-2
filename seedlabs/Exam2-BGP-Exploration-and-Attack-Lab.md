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

![11](./exam2/11.png)
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

![image](https://user-images.githubusercontent.com/4716254/200151339-66a91046-8363-4cb6-843b-dfd03b422d52.png)

##### step 2 Connect AS-180 with AS-2 and AS-3

AS - 180 :  

![2](./exam2/2.png)

AS - 2 : 

![3](./exam2/3.png)

At this point, the host connected to AS-2 can be pinged:

![image](https://user-images.githubusercontent.com/4716254/200151497-52641730-3c4c-4e51-9e8a-14f6cd692aa7.png)

### 4 Task 2: Transit Autonomous System
#### 4.1 Task 2.a: Experimenting with IBGP

Ping 10.164.0.71 on any host of AS-162 to observe the traffic path. At this time, the ibgp3 of the 3/r103 router is turned off, and the icmp path is interrupted. At this time, the routing information forwarded by 10.103.0.3 will be lost in the routing table of the 162/router0 router.

![image](https://user-images.githubusercontent.com/4716254/200151589-ec8c6dd1-17a5-483c-a013-a1d17c862769.png)

#### 4.2 Task 2.b: Experimenting with IGP

Use the command disable ospf1 to turn off the opsf protocol of the 3/r103 router, and the icmp path is cut off.

![image](https://user-images.githubusercontent.com/4716254/200151613-17667106-2ec8-4651-8c49-4eebc6b2784e.png)


#### 4.3 Task 2.c: Configuring AS-5
Modify the configuration file of AS-5, a total of 3, must check. After changing the configuration, AS-153\AS-160\AS-171 can be switched on.
Modify the configurations of AS-5/r103 and AS-3/r103 so that they can communicate with each other in a peer-to-peer manner.

### 5 Task 3: Path Selection

First check the routing table leading to 10.161.0.0/24 in the BGP router of AS-150:

![image](https://user-images.githubusercontent.com/4716254/200151680-8bfde7a5-c7df-4e91-8ffe-13892e311b27.png)

It can be seen that there are two routing paths to 10.161.0.0/24. They pass through AS-2 and AS-3 respectively. When forwarding routes, the first path will be selected for forwarding. This is because the priorities of the two paths are the same. , so the forwarding with the shorter AS path is preferred

![image](https://user-images.githubusercontent.com/4716254/200151692-03cdc577-2e61-42dc-b4df-cc5b1f909098.png)

![image](https://user-images.githubusercontent.com/4716254/200151696-03fefb59-ec75-4d26-b906-af7fa7003215.png)

Next, modify the routing configuration of AS-150 so that the traffic of AS-150 is forwarded through AS-3, and AS-2 is only used as a backup link. Without modifying the routing configuration, traffic destined for 10.152.0.0/24 will be forwarded by AS-2:

![image](https://user-images.githubusercontent.com/4716254/200151711-e465ca48-2885-483a-a228-95d9dbaa1d12.png)

Since the local-preference is preferred when routing, you can adjust the configuration settings.

![image](https://user-images.githubusercontent.com/4716254/200151738-029a194f-f08a-4ff2-9732-669cee6e6c4a.png)


### 6 Task 4: IP Anycast
Anycast (anycast), similar to "throwing hydrangea", a member sends a message to a group of members, and the DNS server adopts this technology.
        
Ping 10.190.0.100 on 10.156.0.71 and 10.160.0.72, and find that the icmp packets of the two hosts have been sent to different destination hosts.

![image](https://user-images.githubusercontent.com/4716254/200151755-bd9f8eb3-f44e-4dff-933d-86addc7a7c6f.png)

The implementation mechanism of anycast is that the router does not care about the specific location of the destination host (even if there are multiple), but only cares about the path to the host. The two 10.190.0.100s respectively inform AS-3 and AS-4 of their own locations, and AS-3 and AS-4 spread out. After other routers receive the routing information, they will select the optimal path for forwarding according to the routing algorithm. , there is only one forwarding path, so the message can only reach a certain host in 10.190.0.100.

### 7 Task 5: BGP Prefix Attack

Principle: longest route matching principle

#### 7.1 Task 5.a. Launching the Prefix Hijacking Attack from AS-161

Modify the configuration information of AS-161 so that all traffic to AS-154 is transferred to AS-161. The subnet in the configuration needs to cover the entire 10.154.0.0/24 :

![4](./exam2/4.png)

The effect is as follows:

![image](https://user-images.githubusercontent.com/4716254/200151891-bc7fbfb4-3812-481f-a7cf-ab57eadf492b.png)


#### 7.2 Task 5.b. Fighting Back from AS-154

Modify the AS-154 configuration so that it can grab its own traffic:

![5](./exam2/5.png)

![image](https://user-images.githubusercontent.com/4716254/200151942-5601ea0f-b630-4891-a67d-bce1d2ade7c7.png)


#### 7.3 Task 5.c. Fixing the Problem at AS-3

Since AS-3 is the only provider of AS-161, AS-3 can modify its own configuration to fix the wrong route:

![6](./exam2/6.png)

The configuration of AS-154 has been rolled back here, and it can be found that the traffic is still sent to AS-154 correctly.

![image](https://user-images.githubusercontent.com/4716254/200152083-a503c40a-4e72-4625-b20f-ba8da159ab34.png)


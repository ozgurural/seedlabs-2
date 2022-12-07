# SEEDlabs: Exam3 - Packet Sniffing and Spoofing Lab

#### Ozgur Ural
#### Student ID: 2564455

### 3 Lab Task Set 1: Using Scapy to Sniff and Spoof Packets

#### 3.1 Task 1.1: Sniffing Packets

First, I install Scapy:
```sh
sudo pip3 install scapy
```

###### Task 1.1A:
This task wants the following program to be ran with root privilege to show it can capture packets, and then ran again without root privilege to see what happens:

![image](https://user-images.githubusercontent.com/4716254/206060394-7d8bba46-c4d7-43da-b414-d417308851ab.png)

The program will sniff for ICMP (Internet Control Message Protocol) packets and print information about the packets to the terminal.

First, I make it executable, then, I run it with root privilege:

```sh
chmod a+x sniffer.py
sudo ./sniffer.py
```


When I first run it, nothing happens. This is because there are no ICMP packets being sent on my network. Opening up another terminal and pinging a website will cause ICMP packets to be sent.Then I attempt the same process again but without root privilege:

As soon as I run sniffer.py I get a permission error.

###### Task 1.1B:

The first filter was actually already accomplished in Task 1.1A, so I will skip to the second filter. I used a second virtual machine so that I can have the python program filter for that machine’s IP address. 

I edit the sniffer.py program to use ‘tcp and src host 10.0.2.4 and dst port 23’ to filter for only tcp packets coming from host 10.0.2.4 and heading to any IP’s port 23:

![image](https://user-images.githubusercontent.com/4716254/206061419-78ca169e-7196-435d-b2d1-66ddf50062e6.png)

I run this with root privilege and attempt to ping to see what happens:
 No packets are captured.

I try the same thing on the other virtual machine with IP address 10.0.2.4:

Packets were captured. This means that the filter worked and only TCP packets from 10.0.2.4 being sent to port 23 are captured; the rest were ignored.

The last filter I need to implement is one that will only capture packets coming from or going to a particular subnet. I will choose 128.115.0.0/16 as the subnet. I edit the sniffer.py program as follows:

![image](https://user-images.githubusercontent.com/4716254/206061656-d13173f4-8dee-4d1b-b7ae-b7b7df65e96d.png)

Running sniffer.py with root privilege, I try sending a TCP packet with the nc command to an IP that isn’t part of the subnet, nothing happens.

I try again, but this time I send it to an IP address that is part of the subnet. Packets are being captured. The filter is working.


#### 3.2 Task 1.2: Spoofing ICMP Packets

In this task, I need to make a Python program that uses Scapy to create a spoofed ICMP echo request packet with an arbitrary source IP address and send it to another virtual machine on my network.

I will make the arbitrary source IP address 1.2.3.4 and the destination address 10.0.2.4 (this is the IP address of my Server VM). Here is the program:

![image](https://user-images.githubusercontent.com/4716254/206061809-576a217e-da81-4dad-ae00-5c733e1597f3.png)

The program first creates an IP object and sets the destination and source IP addresses. Then it creates an ICMP object. The default type for ICMP objects in Scapy is echo request, so that doesn’t need to be explicitly set.

Next, the program creates the packet by using ip/icmp (this sets the ICMP object as the IP object’s payload). Finally, it sends the packet out.

I use the tcpdump command to listen for ICMP packets on the network. It will write information about any packets of that type to a file called ‘packets’ in the ‘tmp’ directory.

In a second terminal window, I run the spoofing.py program with root privilege. I ctr+c out of the tcpdump listener and use Wireshark to open the /tmp/packets file:

![image](https://user-images.githubusercontent.com/4716254/206061935-87ef9d20-20ac-46a5-a107-52cb75caba7f.png)

The first ICMP packet that tcpdump captured was the echo request that the spoofing.py program sent. The second ICMP packet captured was the reply sent back from 10.0.2.4 (the Server VM).

The source for the request is 1.2.3.4 and the destination for the reply is also 1.2.3.4. This means that the spoofing.py program successfully spoofed a ICMP packet and assigned it an arbitrary source IP address.



#### 3.3 Task 1.3: Traceroute

The goal of this task is to create a version of traceroute using Scapy. The program needs to repeatedly send out packets (I will use ICMP packets) with Time-To-Live (TTL) value starting at 1.

I will make the maximum number of hops 255. This means that if the packet fails to reach its destination by the time its TTL has been incremented all the way to 255, the program will stop.

I used the following program:

![image](https://user-images.githubusercontent.com/4716254/206062158-8c57fa67-053b-4c81-876a-05174ec0bae7.png)

I test the trace.py program out by having it attempt to go to a website. It does so successfully after 13 hops.

I next try a random IP address (1.2.3.4). After six hops, the program is no longer receiving a reply.





#### 3.4 Task 1.4: Sniffing and-then Spoofing

For this task I will be using two virtual machines on my LAN: Attacker (IP 10.0.2.15) and Server (IP 10.0.2.4):

Here is the python program:

![image](https://user-images.githubusercontent.com/4716254/206062414-713891f6-be62-4f64-afe6-4c3df35f1cbf.png)

I try to ping an IP address that I know isn’t alive (1.2.3.4) on the Server machine to see what happens when the sniffAndSpoof.py program isn’t running, we get no replies.

I now run the sniffAndSpoof.py program on the Attacker machine and run the same ping command on the Server machine. I begin getting replies on the Server machine. It says the replies are from IP 1.2.3.4, but I know that isn’t true because that is a dead IP address as shown above. The replies are actually coming from the sniffAndSpoof.py program that is running on the Attacker machine. The program is working.

### 4 Lab Task Set 2: Writing Programs to Sniff and Spoof Packets

#### 4.1 Task 2.1: Writing Packet Sniffing Program

##### Task 2.1A: Understanding How a Sniffer Works

##### Task 2.1B: Writing Filters

##### Task 2.1C: Sniffing Passwords

#### 4.2 Task 2.2: Spoofing

##### Task 2.2A: Write a spoofing program

##### Task 2.2B: Spoof an ICMP Echo Request

#### 4.3 Task 2.3: Sniff and then Spoof

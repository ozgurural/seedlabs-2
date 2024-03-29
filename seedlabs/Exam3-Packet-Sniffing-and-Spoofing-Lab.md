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

I run this with root privilege and attempt to ping to see what happens, no packets are captured. I try the same thing on the other virtual machine with IP address 10.0.2.4:Packets were captured. This means that the filter worked and only TCP packets from 10.0.2.4 being sent to port 23 are captured; the rest were ignored.

The last filter I need to implement is one that will only capture packets coming from or going to a particular subnet. I will choose 128.115.0.0/16 as the subnet. I edit the sniffer.py program as follows:

![image](https://user-images.githubusercontent.com/4716254/206061656-d13173f4-8dee-4d1b-b7ae-b7b7df65e96d.png)

Running sniffer.py with root privilege, I try sending a TCP packet with the nc command to an IP that isn’t part of the subnet, nothing happens. I try again, but this time I send it to an IP address that is part of the subnet. Packets are being captured. The filter is working.

#### 3.2 Task 1.2: Spoofing ICMP Packets

In this task, I need to make a Python program that uses Scapy to create a spoofed ICMP echo request packet with an arbitrary source IP address and send it to another virtual machine on my network.

I will make the arbitrary source IP address 1.2.3.4 and the destination address 10.0.2.4. Here is the program:

![image](https://user-images.githubusercontent.com/4716254/206061809-576a217e-da81-4dad-ae00-5c733e1597f3.png)

The program first creates an IP object and sets the destination and source IP addresses. Then it creates an ICMP object. The default type for ICMP objects in Scapy is echo request, so that doesn’t need to be explicitly set. Next, the program creates the packet by using ip/icmp. Finally, it sends the packet out.

I use the tcpdump command to listen for ICMP packets on the network. It will write information about any packets of that type to a file called ‘packets’ in the ‘tmp’ directory. In a second terminal window, I run the spoofing.py program with root privilege. I copy out of the tcpdump listener and use Wireshark to open the /tmp/packets file:

![image](https://user-images.githubusercontent.com/4716254/206061935-87ef9d20-20ac-46a5-a107-52cb75caba7f.png)

The first ICMP packet that tcpdump captured was the echo request that the spoofing.py program sent. The second ICMP packet captured was the reply sent back from 10.0.2.4. The source for the request is 1.2.3.4 and the destination for the reply is also 1.2.3.4. This means that the spoofing.py program successfully spoofed a ICMP packet and assigned it an arbitrary source IP address.

#### 3.3 Task 1.3: Traceroute

The goal of this task is to create a version of traceroute using Scapy. The program needs to repeatedly send out packets with Time-To-Live(TTL) value starting at 1.
I make the maximum number of hops 255. This means that if the packet fails to reach its destination by the time its TTL has been incremented all the way to 255, the program will stop.

I used the following program:

![image](https://user-images.githubusercontent.com/4716254/206062158-8c57fa67-053b-4c81-876a-05174ec0bae7.png)

I test the trace.py program out by having it attempt to go to a website. It does so successfully after 13 hops. I next try a random IP address (1.2.3.4). After six hops, the program is no longer receiving a reply.

#### 3.4 Task 1.4: Sniffing and-then Spoofing

For this task I will be using two virtual machines on my LAN: Attacker (IP 10.0.2.15) and Server (IP 10.0.2.4):

Here is the python program:

![image](https://user-images.githubusercontent.com/4716254/206062414-713891f6-be62-4f64-afe6-4c3df35f1cbf.png)

I try to ping an IP address that I know isn’t alive (1.2.3.4) on the Server machine to see what happens when the sniffAndSpoof.py program isn’t running, we get no replies.

I now run the sniffAndSpoof.py program on the attacker machine and run the same ping command on the server machine. I begin getting replies on the server machine. It says the replies are from IP 1.2.3.4, but I know that isn’t true because that is a dead IP address as shown above. The replies are actually coming from the sniffAndSpoof.py program that is running on the Attacker machine. The program is working.

### 4 Lab Task Set 2: Writing Programs to Sniff and Spoof Packets

#### 4.1 Task 2.1: Writing Packet Sniffing Program

##### Task 2.1A: Understanding How a Sniffer Works

Create a program to print the source IP and destination IP address of the captured packet

![image](https://user-images.githubusercontent.com/4716254/206075719-dff9dcc1-78db-478e-a3c4-6f3029917a37.png)

Pinged website in another terminal and observed that the sent package appears in the running result.

Question 1: Start pcap to listen to the network card. Then, compile BPF filter and set the filter. Set sniffing The processing function, and finally close the sniffing. 

Question 2: Sniffing packets is a high-privilege operation, because it involves privacy and security-related issues. If ordinary users can also sniff data packets, then he can steal other people's privacy, even steal account passwords and so on. Run the program without root privileges. The first step of monitoring the network card fails when there is no permission.

Question 3: Using the promiscuous mode can monitor the data packets of other machines in the same network segment, but not when it is turned off. As shown below, enable the promiscuous mode on the terminal through the sudo ifconfig enp0s3 promisc command line, and listen to the data packets of ping a website from another machine (10.0.2.4) in this network segment, but it cannot be sniffed after it is turned off. 

```sh
sudo ifconfig enp0s3 promisc
ping google.com
```

![image](https://user-images.githubusercontent.com/4716254/206267809-5bd759ca-eb4e-4757-9f6d-fd401ccb5897.png)

##### Task 2.1B: Writing Filters

Capture ICMP packets between two specific hosts. 
The filters used are icmp and src host 10.0.2.15 and dst host 220.181.38.148. Only capture ICMP packets sent from 10.0.2.15 to 220.181.38.148. The results are as follows. It can be seen that all ICMP packets are sent from 10.0.2.15 to 220.181.38.148, and there are no other types of packets.

![image](https://user-images.githubusercontent.com/4716254/206267525-04ad1a74-6024-43af-805a-2f478d0c141d.png)

Capture TCP packets whose destination port number is in the range of 10 to 100. The filter used is tcp and dst portrange 10-100. The result is as follows: access the website with a browser: the packet of 111 does not appear in the result.

![image](https://user-images.githubusercontent.com/4716254/206076615-9fef4800-bae9-4ce1-8fe6-b299bc92a290.png)

##### Task 2.1C: Sniffing Passwords

Use sniffing to capture the password in the telnet protocol, the code is as follows:

![image](https://user-images.githubusercontent.com/4716254/206076701-1a5e66ee-0906-444b-ad53-a34d78f40e6a.png)

Then use telnet 10.9.0.5, and enter the account password for remote login. The sniffed passwords are as follows:

![image](https://user-images.githubusercontent.com/4716254/206076961-5c370a8b-ea62-4b27-b5c1-edc0e447cb5e.png)
![image](https://user-images.githubusercontent.com/4716254/206077819-00158e60-98d6-487c-a617-2457e6d5587f.png)
![image](https://user-images.githubusercontent.com/4716254/206077840-5d79a4f6-d2f8-488e-9fa7-1c2e7fdc2e32.png)
![image](https://user-images.githubusercontent.com/4716254/206077855-7d193010-803b-4f9d-8b04-b6693fbcec5e.png)
![image](https://user-images.githubusercontent.com/4716254/206077867-fc2e51d2-3fd3-473c-917d-59129460dc98.png)

The input password is dees, and the user name input by telnet is the same.

#### 4.2 Task 2.2: Spoofing

##### Task 2.2A: Write a spoofing program

Create a task22a.c to forge UDP packets, see the code in the compressed package.  Use gcc -o task22a task22a.c spoof.c -lpcap to compile, then sudo ./task22a to run the program, use wireshark to view the background, you can see forged UDP packets.

![image](https://user-images.githubusercontent.com/4716254/206268227-e6845b45-8f06-4d71-9b71-29689807b701.png)

##### Task 2.2B: Spoof an ICMP Echo Request

Forged ICMP Echo request, forged code task22b.c, where the source IP 10.9.0.6 is the IP of another container in the LAN

![image](https://user-images.githubusercontent.com/4716254/206078200-4aab7a49-6681-4d27-94ac-7e47db9b300d.png)

Use gcc -o task22b task22b.c spoof.c checksum.c -lpcap to compile, run sudo ./task22b, check the wireshark in the background, as follows, you can see that the source IP we sent is 10.9.0.6, and the destination IP is 8.8. 8.8 ICMP packets, and there are replies:

![image](https://user-images.githubusercontent.com/4716254/206268435-454de613-e9bc-4429-b32b-b6f5beece751.png)

Question 4: Change the length set in the code to the following, run it to know that it is 28B, modify the length to 10B, wireshark did not capture the packet, indicating that it was not sent.

![image](https://user-images.githubusercontent.com/4716254/206078277-c1f19672-38a0-4eeb-a092-f73399d44287.png)

Modify it to 1000B, the result is as follows, you can see that it was sent out normally, and the response was received. Indicates that the length can be increased but cannot be decreased.

![image](https://user-images.githubusercontent.com/4716254/206078315-e1c6cb6c-d2ce-4476-8cd3-f923068a1872.png)

Question 5: You do not need to calculate the checksum for the IP header, but you need to calculate the checksum for the ICMP header. 

Question 6: Because being able to read and send packets arbitrarily means a great security risk, so root privilege is required . The results of running with normal permissions are as follows:

```sh
./task22b
sock: -1
```

#### 4.3 Task 2.3: Sniff and then Spoof

Create a task23.c, part of the code is as follows:

![image](https://user-images.githubusercontent.com/4716254/206078654-8544d78b-8c0a-4b57-99bd-ec15816a8537.png)

Use gcc -o task23 task23.c checksum.c spoof.c -lpcap to compile the program, sudo ./task23 to run. Close the network, use another machine to ping 1.1.1.1, this machine runs the above program, the result is as follows:

```sh
ping 1.1.1.1
```

![image](https://user-images.githubusercontent.com/4716254/206269315-12501a9a-72bc-44fd-bb4c-0dc70ae2a4b1.png)

```sh
sudo ./task23
```

![image](https://user-images.githubusercontent.com/4716254/206269420-6f7f9dc6-740c-4734-8444-c1f183c27d46.png)

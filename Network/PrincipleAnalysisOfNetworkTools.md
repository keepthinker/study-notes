# LVS
负载均衡，通过NAT转换，路由外部请求到子网，相当于针对传输层的第四层路由器。

不处理子网节点宕机，移除节点的工作，需要其他工具协助。

A Linux Virtual Server (LVS) is a cluster of servers which appears to be one server to an outside client. This apparent single server is called here a "virtual server". The individual servers (realservers) are under the control of a director (or load balancer), which runs a Linux kernel patched to include the ipvs code. The ipvs code running on the director is the essential feature of LVS. Other user level code is used to manage the LVS (set rules for services handled, handle failover). The director is basically a layer 4 router with a modified set of routing rules (i.e. connections do not originate or terminate on the director, it doesn't send ACKs etc, it's just a router).

When a new connection is requested from a client to a service provided by the LVS (e.g. httpd), the director will choose a realserver for the client. From then, all packets from the client will go through the director to that particular realserver. The association between the client and the realserver will last for only the life of the tcp connection (or udp exchange). For the next tcp connection, the director will choose a new realserver (which may or may not be the same as the first realserver). Thus a web browser connecting to an LVS serving a webpage consisting of several hits (images, html page), may get each hit from a separate realserver.

Management of the LVS is through the user space utility ipvsadm and schedulers, which is used to add/removed realservers/services and to handle failout. LVS itself does not detect failure conditions; these are detected by external agents, which then update the state of the LVS through ipvsadm.


# traceroute
In computing, traceroute is a computer network diagnostic tool for displaying the route (path) and measuring transit delays of packets across an Internet Protocol (IP) network.

## Implementation
On **Unix-like operating systems**, traceroute sends, **by default, a sequence of User Datagram Protocol (UDP) packets**, with destination port numbers ranging from 33434 to 33534; the implementations of traceroute shipped with Linux, FreeBSD, NetBSD, OpenBSD, DragonFly BSD, and macOS include an option to use **ICMP(Internet Control Message Protocol)** Echo Request packets (-I), or any arbitrary protocol (-P) such as UDP, **TCP** using TCP SYN packets, or ICMP. In Windows, traceroute sends ICMP echo requests instead of UDP packets.

The time-to-live (TTL) value, also known as hop limit, is used in determining the intermediate routers being traversed towards the destination. Traceroute sends packets with TTL values that gradually increase from packet to packet, starting with TTL value of one. Routers decrement TTL values of packets by one **when routing and discard packets whose TTL value has reached zero, returning the ICMP(Internet Control Message Protocol) error message ICMP Time Exceeded.** For the first set of packets, the first router receives the packet, decrements the TTL value and drops the packet because it then has TTL value zero. The router sends an **ICMP Time Exceeded message** back to the source. The next set of packets are given a TTL value of two, so the first router forwards the packets, but the second router drops them and replies with ICMP Time Exceeded. Proceeding in this way, traceroute uses the returned ICMP Time Exceeded messages to build a list of routers that packets traverse, **until the destination is reached and returns an ICMP Echo Reply message.**

The sender expects a reply within a specified number of seconds. If a packet is not acknowledged within the expected interval, an asterisk is displayed. The Internet Protocol does not require packets to take the same route towards a particular destination, thus hosts listed might be hosts that other packets have traversed. If the host at hop #N does not reply, the hop is skipped in the output.

example:
```c
keepthinker@keepthinker-Lenovo-IdeaPad-Y470:~$ traceroute www.baidu.com
traceroute to www.baidu.com (14.215.177.39), 30 hops max, 60 byte packets
 1  boxapi.sutongwang.com (192.168.8.1)  6.742 ms  6.692 ms  10.604 ms
 2  192.168.1.1 (192.168.1.1)  15.741 ms  32.181 ms  35.022 ms
 3  100.64.0.1 (100.64.0.1)  45.546 ms  45.545 ms  45.514 ms
 4  61.146.244.201 (61.146.244.201)  55.980 ms  55.978 ms  55.897 ms
 5  219.133.207.85 (219.133.207.85)  61.595 ms *  76.842 ms
 6  * * *
 7  113.96.4.209 (113.96.4.209)  44.602 ms 106.96.135.219.broad.fs.gd.dynamic.163data.com.cn (219.135.96.106)  16.518 ms  16.541 ms
 8  14.215.32.130 (14.215.32.130)  21.970 ms 98.96.135.219.broad.fs.gd.dynamic.163data.com.cn (219.135.96.98)  44.371 ms 14.29.121.194 (14.29.121.194)  44.228 ms
 9  14.29.121.206 (14.29.121.206)  44.247 ms * *
10  * * *
11  * * *
12  * * *
13  * * *
14  * * *
15  * * *
16  * * *
17  * * *

```

# Ping
Ping operates by sending **Internet Control Message Protocol (ICMP) echo request packets** to the target host and waiting for an **ICMP echo reply.** The program reports errors, packet loss, and a statistical summary of the results, typically including the minimum, maximum, the mean round-trip times, and standard deviation of the mean.
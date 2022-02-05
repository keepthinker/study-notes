# 链路层
## Ethernet
### Ethernet(IEEE 802.3) frame format.

(Preamble, SDF)DST, SRC, length or type , [P/Q tag|Other tags] Upper-Layer Protocol Payload [pad], FCS (Carrier Extension).

- **Preamble(7 bytes)**: The Ethernet frame begins with a Preamble area used by the receiving interface's circuitry to determine when a frame is arriving and to determine the amount
of time between encoded bits (called clock recovery)
- **SFD(1 byte)**: start frame deliminater, fixed value 0xAB.
- **DST(6 bytes)**: destination MAC address
- **SRC(6 bytes)**: source MAC address
- **Type or Length(2 bytes)**: Type field that doubles as a Length field. Ordinarily, it identifies the type of data that follows the header. Popular values used with TCP/IP networks include IPv4 (0x0800), IPv6 (0x86DD), and ARP (0x0806)
- **Upper-Layer Protocol Payload**: This is the area where higher-layer PDUs such as IP datagrams are placed.
- **FCS(4 bytes)**: frame check sum. It provides
an integrity check on the frame. The Cyclic Redundancy Check (CRC) field at the end includes 32 bits and is sometimes known as the IEEE/ANSI standard CRC32 [802.3-2008]

#### Frame Size
There is both a minimum and a maximum size of Ethernet frames. The minimum is 64 bytes, requiring a minimum data area (payload) length of 46 bytes (no tags). In cases where the payload is smaller, pad bytes (value 0) are appended to the end of the payload portion to ensure that the minimum length is enforced.

The maximum frame size of conventional  Ethernet is 1518 bytes (including the 4-byte CRC and 14-byte header), but the more recent standard extended this size to 2000 bytes. Most systems today use the 1500-byte MTU size for Ethernet. 

### MTU and Path MTU
There is a limit on the size of the frame available for carrying the PDUs of higher-layer protocols in many link-layer networks such as Ethernet. This usually limits the number of payload bytes to about 1500 for Ethernet and often the same amount for PPP in order to maintain compatibility with Ethernet. This characteristic of the link layer is called the maximum transmission
unit (MTU).

When two hosts communicate across multiple networks, each link can have a different MTU. The minimum MTU across the network path comprising all of the links is called the path MTU

PMTU discovery is used to determine the path MTU at a point in time and is required of IPv6 implementations

## Tunneling
In some cases it is useful to establish a **virtual link** between one computer and
another across the Internet or other network. VPNs, for example, offer this type of
service. Tunneling, generally speaking, is the idea of carrying lower-layer traffic in higher-layer (or equal-layer) packets.

# ARP
The operating system software (i.e., the Ethernet driver) must know the destination's hardware address to send data directly. For TCP/IP networks, the Address Resolution Protocol (ARP) [RFC0826] provides a dynamic mapping between IPv4 addresses and the hardware addresses used by various network technologies. ARP is used with IPv4 only; IPv6 uses the Neighbor Discovery Protocol, which is incorporated into ICMPv6.

When an Ethernet frame containing an IP datagram is sent from one host on a LAN to another, it is the 48-bit Ethernet address that determines to which interface(s) the frame is destined

RP is a generic protocol, in the sense that it is designed to support mapping between a wide variety of address types. In practice, however, it is almost always used to map between 32-bit IPv4 addresses and Ethernet-style 48-bit MAC addresses. ARP provides a **dynamic mapping** from a network-layer address to a corresponding hardware address.

ARP operates only when reaching those systems on the same IP subnet.

ARP例子：
1. 应用访问浏览器一个网址，如果有域名，则转换成IPv4。
2. 应用请求建立TCP连接。
3. 假设目标IPv4地址与本机处于同一个子网。
4. 假设Ethernet兼容的网址被使用在该子网，那么发送方需要转换IPv4成48 bit以太网类型地址（MAC地址）。需要使用到ARP进行转换。ARP一般情况下使用广播网络来工作，在该网络，链路层可以发送消息到所有连接的设备。
5. ARP发起携带目标IP的请求到每个共享的链路层的主机询问MAC地址。
6. 与目标IP匹配的主机返回ARP回复。
7. ARP请求方接收到回复后，开始传输被以太帧封装的数据到目标机器。

ARP is used in multi-access link-layer networks running IPv4, where each host has its own primary hardware address.

## ARP Format
Hard Type(2 bytes), Prot Type(2), Hard Size(1), prot Size(1), Op(1), Sender's Hardware Address(6), Sender's Protocol Address(4), Target Hardware Address(6), Target Protocol Address(4)

For ARP requests, the special Ethernet destination address of ff:ff:ff:ff:ff:ff (all 1 bits) means the broadcast address—all Ethernet interfaces in the same broadcast domain receive these frames.

- The **first four fields**  following the Length/Type field specify the types and sizes of the final four fields.
- The **Hardware address**  is MAC, physical, or link-layer address
(or Ethernet address when the network in use is based on the IEEE 802.3/Ethernet series of specifications).
- The **Hard Type** field specifies the type of hardware
address. Its value is 1 for Ethernet.
- The **Prot Type** field specifies the type of protocol
address being mapped. Its value is 0x0800 for IPv4 addresses. 
- The next two 1-byte fields, **Hard Size** and **Prot Size**, specify the sizes, in bytes, of the hardware addresses and the protocol addresses.
- The **Op** field specifies whether the operation is an ARP request (a value of 1), ARP reply (2), RARP request (3), or RARP reply (4)

The next four fields that follow are the Sender's **Hardware Address (an Ethernet MAC address in this example)**, the Sender's **Protocol Address(an IPv4 address)**, the **Target Hardware (MAC/Ethernet) Address**, and the **Target Protocol (IPv4) Address**

when a system receives an ARP request addressed to it, in addition to sending the ARP reply, it also saves the requestor's hardware address and IPv4 address in its own ARP cache.

## ARP Linux参数
**net.ipv4.neigh.default.gc_stale_time**:
> gc_stale_time (since Linux 2.2).
Determines how often to check for stale neighbor entries. When a neighbor entry is considered stale, it is resolved again before sending data to it. Defaults to 60 seconds.

/proc/sys/net/ipv4/neigh/default/gc_thresh1

**net.ipv4.neigh.default.gc_thresh1**: 
>  gc_thresh1 (since Linux 2.2). The minimum number of entries to keep in the ARP cache. The garbage collector will not run if there are fewer than this number of entries in the cache. Defaults to 128.

**/proc/sys/net/ipv4/neigh/default/gc_thresh2**
> gc_thresh2 (since Linux 2.2). The soft maximum number of entries to keep in the ARP cache. The garbage collector will allow the number of entries to exceed this for 5 seconds before collection will be performed. Defaults to 512.
gc_thresh3 (since Linux 2.2)


**/proc/sys/net/ipv4/neigh/default/gc_thresh3**
>  gc_thresh3 (since Linux 2.2). The hard maximum number of entries to keep in the ARP cache. The garbage collector will always run if there are more than this number of entries in the cache. Defaults to 1024.




# 系统配置：DHCP与自动配置

# NAT
通过IP和端口区分内网主机对应哪个包方法，来解决全球IP不够用问题。

保护内网机器IP不对外暴露，防探测，起到一定安全作用。


## Examples
Setting up a virtual HTTP server with two real servers:
<pre>
ipvsadm -A -t 192.168.0.1:80 -s rr
ipvsadm -a -t 192.168.0.1:80 -r 172.16.0.1:80 -m
ipvsadm -a -t 192.168.0.1:80 -r 172.16.0.2:80 -m
</pre>
The first command assigns TCP port 80 on IP address 192.168.0.1 to the virtual server. The chosen scheduling algorithm for load balancing is round-robin (-s rr). The second and third commands are adding IP addresses of real servers to the LVS setup. The forwarded network packets shall be masked (-m).

Querying the status of the above configured LVS setup:
<pre>
ipvsadm -L -n
IP Virtual Server version 1.0.8 (size=65536)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  192.168.0.1:80 rr
  -> 172.16.0.2:80                Masq    1      3          1
  -> 172.16.0.1:80                Masq    1      4          0
</pre>



# Select
## Linux Implementation
## Select/Poll
 The application generates an array of fds that it is interested in. Then the application make a system call to the Kernel. The Kernel will then copy the array of fds from user space and then went through every single one of them to see whether there are any events available with file's poll() operation. After that, select will simply generate a bit array and copy it back to the user space, where poll will directly manipulate the pollfd struct in the user space, without copying back. Time complexity are both O(n).
 
 The biggest difference between epoll and traditional I/O multiplexing mechanisms is that, instead of building and passing a giant array of file descriptors into the Kernel every time, the application simply acquires an epoll instance and register file descriptors onto it. And rather than polling a whole array of file descriptors, the epoll instance monitors registered file descriptors and "report" events to the application when being asked.


## Epoll vs Select/Poll
Epoll is meant to replace the older POSIX select and poll system calls, to achieve better performance in more demanding applications, where the number of watched file descriptors is large (unlike the older system calls, which operate in O(n) time, epoll operates in O(1) time[2]).

## Epoll
Epoll uses a very commonly used kernel data structure - Red–black tree (abbreviated as RB-Tree), to keep track of all file descriptors that are currently being monitored by a certain epoll instance. The root of the RB-Tree is the rbr member of struct eventpoll, and is initialized within the ep_alloc() function.


# DNS
DNS is a distributed client/server networked database that is used by TCP/IP applications to map between host names and IP addresses。

The DNS protocol consists of two main parts: a query/response protocol used for performing queries  against the DNS for particular names, and another protocol for name servers to exchange database  records (zone transfers).

From an application's point of view, access to the DNS is through an application library called a  resolver. In general, an application must convert a host name to an IPv4 and/or IPv6 address before it  can ask TCP to open a connection or send a unicast datagram using UDP.

DNS messages are typically encapsulated in a UDP/IPv4 datagram and are limited to 512 bytes in size  unless TCP and/or EDNS0 is used. 

Full zone transfers use TCP as they can be large. Incremental zone transfers, where only the updated  entries are transferred, may use UDP at first but switch to TCP if the response is too large, just  like a conventional query.


# Head-of-line blocking
Head-of-line blocking (HOL blocking) in computer networking is a performance-limiting phenomenon that occurs when a line of packets is held up by the first packet.

A switch may be composed of buffered input ports, a switch fabric and buffered output ports. If first-in first-out (FIFO) input buffers are used, only the oldest packet is available for forwarding. More recent arrivals cannot be forwarded if the oldest packet cannot be forwarded because its destination output is busy. 

Without HOL blocking, the new arrivals could potentially be forwarded around the stuck oldest packet to their respective destinations. The phenomenon can have severe performance-degrading effects in input-buffered systems.


## Virtual output queueing
Virtual output queueing (VOQ) is a technique used in certain network switch architectures where, rather than keeping all traffic in a single queue, separate queues are maintained for each possible output location. It addresses a common problem known as head-of-line blocking.


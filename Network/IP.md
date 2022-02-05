# 简介
All TCP, UDP, ICMP, and IGMP data gets transmitted as IP datagrams. IP provides a **best-effort, connectionless** datagram delivery service. By “best-effort” we mean there are no guarantees that an IP datagram gets to its destination successfully. IPv4 and IPv6 both use this basic best-effort delivery model.

When something goes wrong, such as a router temporarily running out of buffers, IP has a simple **error-handling algorithm: throw away some data** (usually the last datagram that arrived)

**what does connectionless mean**
- IP does not maintain any connection state information about related datagrams within the network elements (i.e., within the routers); each datagram is handled independently from all other others. 
- This also means that IP datagrams can be delivered out of order. 
- They may be duplicated in transit.
- They may have their data altered as the result of errors.

# IP格式
## IPv4格式
Version(4 bits), IHL(4), DSField(6), ECN(2), Total Length(16), Identification(16), Flags(3), Fragment Offset(13), Time-to-Live(TTL)(8), Protocol(8), Header Checksum(16), Source IP Address(32), Destination IP Address(32), Options(If Any), IP Data(If Any)

- **IHL(Internet Header Length)**: The header is of variable size, limited to fifteen 32-bit words (60
bytes) by the 4-bit IHL field.  A typical IPv4 header contains **20 bytes** (no options)
- **Header Checksum**: A header checksum helps ensure that the fields in
the header are delivered correctly to the proper destination but does not protect the data.

- **Total Length**: Is the total length of the IPv4 datagram in bytes. Because this is a 16-bit field, the maximum size of an IPv4  datagram (including header) is 65,535 bytes. When an IPv4 datagram is fragmented into multiple smaller fragments, each of which itself is an independent IP datagram, the  Total Length field reflects the length of the particular fragment.

- **Identification**: Helps indentify each datagram sent by an IPv4 host. To ensure that the fragments of one datagram are not confused with those of another, the sending host normally increments an internal counter by 1 each time a datagram is sent (from one of its IP addresses) and copies the value of the counter into the IPv4 Identification field.

- **Time to Live(TTL)**: It sets an upper limit on the number of routers through which a datagram can pass. It is initialized by the sender to some value (64 is recommended [RFC1122]) and decremented by 1 by every router  that forwards the datagram. When this field reaches 0, the datagram is thrown away, and the sender is notified with an ICMP message.  In IPv6 the field has been renamed to its de facto use: **Hop Limit**.

- **Protocol**: A number indicating the type of data found in the payload portion of the datagram. It is now understood to identify the encapsulated protocol, which may or not be a transport protocol. For example, other encapsulations are possible, such as IPv4-in-IPv4 (value 4).

- **Header Checksum**: Calculated over the IPv4 header only. This is important to understand because it means that the payload of the IPv4 datagram (e.g., TCP or UDP data) is not checked for correctness by the IP protocol. We shall see that almost all protocols encapsulated in IP (ICMP, IGMP, UDP, and TCP) have a checksum in their own headers to cover their header and data and also to cover certain parts of the IP header they deem important (a form of "layering violation").





## IPv6格式
Version(4 bits), DSfield(6), ECN(2), Flow Label(20), Payload Length(16), Next Header(8), Hop Limit(8),  Source IP Address(128), Destination IP Address(128)

The IPv6 header is of fixed size (**40 bytes**)

- **Next Header**: It generalizes the Protocol field from IPv4. It is used to indicate the type of header following the IPv6 header. This field may contain any values defined for the IPv4 Protocol field, or any of the values associated with the IPv6 extension headers 

- **Version**: It contains the version number of the IP datagram: 4 for IPv4 and 6 for IPv6



- **Payload Length**: This field measures the length of the IPv6 datagram not including the length of the header; extension headers, however, are included in the Payload Length field. With IPv6, however, it is the payload length that is limited to 64KB, not the entire datagram.

- **Hop Limit**: see Time to Live in IPv4.

Type of Service: **DSField**(Differentiated
Services Field), **ECN**(Explicit Congestion Notification). These fields are used for special processing of the datagram when it is forwarded.

**Source IP Address & Destination IP Address**: Every IP datagram contains the Source IP Address of the sender of the datagram and the Destination IP Address of where the datagram is destined. These are 32-bit values for IPv4 and 128-bit values for IPv6, and they usually identify a single interface on a computer, although multicast and broadcast addresses violate this rule. 

A host is not required to be able to receive an IPv4 datagram larger than 576 bytes. (In IPv6 a host must be able to process a datagram at least as large as the MTU of the link to which it is attached, and the minimum link MTU is 1280 bytes.) 

# IP

IP地址包含网络部分与主机部分。

## 分类地址
Class | Address Range | High-Order Bits | Use | Fraction of Total | Number of Nets | Number of Hosts 
---|---|---|---|---|---|---|---
A | 0.0.0.0–127.255.255.255 | 0 | Unicast/special | 1/2 | 128 | 16,777,216
B | 128.0.0.0–191.255.255.255 | 10 | Unicast/special | 1/4 | 16,384 | 65,536
C | 192.0.0.0–223.255.255.255 | 110 | Unicast/special | 1/8 | 2,097,152 | 256
D | 224.0.0.0–239.255.255.255 | 1110 | Multicast | 1/16 | N/A | N/A
E | 240.0.0.0–255.255.255.255 | 1111 | Reserved | 1/16 | N/A | N/A

## 子网地址
为了解决分配英特网和本地网络的网络号分配的不方便，提出了改变网络号与主机号的区分边界的方法来规划网络号。

### 广播地址
用子网掩码的取反与子网里任何主机地址（或子网前缀）做一次或运算。

#### 链路层广播地址
ff:ff:ff:ff:ff:ff

## 特殊IP地址
255.255.255.255 本地网络广播（受限广播）地址，不可被路由器转发，向自己所在网络所有机器广播。

127.0.0.0/8 本地回环地址比如127.0.0.1 ，代表设备的本地虚拟接口IP。

0.0.0.0  在IP数据报中只能用作源IP地址，这发生在当设备启动时但又不知道自己的IP地址情况下。
1. 在服务器中，0.0.0.0指的是本机上的所有IPV4地址，如果一个主机有两个IP地址，192.168.1.1 和 10.1.2.1，并且该主机上的一个服务监听的地址是0.0.0.0,那么通过两个ip地址都能够访问该服务。 
2. 在路由中，0.0.0.0表示的是默认路由，表示”任意IPV4主机”，即当路由表中没有找到完全匹配的路由的时候所对应的路由。

224.0.0.0/4 多播(multicast)地址

### 内网地址
- 10.0.0.0/8
- 172.16.0.0/12 范围既172.16.0.0 ~ 172.31.255.255
- 192.168.0.0/16

### 多播地址
- 224.0.0.0–224.0.0.255 The local network control block is
limited to the local network of the sender; datagrams sent to those addresses are
never forwarded by multicast routers.
- 224.0.1.0–224.0.1.255 Internetwork control; forwarded normally. The internetwork control block is similar to the local network control
range but is intended for control traffic that needs to be routed off the local link.
- 224.0.2.0–224.0.255.255 Ad hoc block I. The first ad hoc block was constructed to hold addresses that did not fall into
either the local or internetwork control blocks. Most of the allocations in this range
are for commercial services


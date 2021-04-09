# Introduction
UDP is a simple, datagram-oriented, transport-layer protocol that preserves message boundaries. 

It does not provide error correction, sequencing, duplicate elimination, flow control, or congestion control. It can provide error detection, and it includes the end-to-end checksum.

This protocol provides minimal functionality itself, so applications using it have a great deal of control over how packets are sent and processed. Applications wishing to ensure that their data is reliably delivered or sequenced
must implement these protections themselves.

Generally, each UDP output operation requested by an application produces exactly one UDP datagram, which causes one IP datagram to be sent. 


# UDP Header
Because IP demultiplexes the incoming IP datagram to a particular transport protocol based on the value of the Protocol field in the IPv4 header or Next Header field in the IPv6 header, this means that the port numbers
can be made independent among the transport protocols. Two completely distinct servers can use the same port number and IP address, as long as they use different transport protocols.

## Format
Source Port Number(2 bytes), Destination Port Number(2 bytes), Length(2 bytes), Checksum(2 bytes)

- **Length**: The length of the UDP header
and the UDP data in bytes. Note that the UDP Length field is redundant; the IPv4 header contains the datagram's total length, and the IPv6 header contains the payload length. 

- **Checksum**: The UDP checksum covers the UDP header, the UDP data, and a pseudo-header . It is not modified in transit (except when it passes through a NAT), With UDP, **the checksum is optional** (although strongly suggested), while with the others it is mandatory. When UDP is used with IPv6, computation and use of the checksum are mandatory because there is no header checksum at the IP layer. UDP (as well as TCP) computes its checksum over a 12-byte pseudo-header derived (solely) from fields in the IPv4 header or a 40-byte pseudo-header derived from fields in the IPv6 header. Upon receipt, a Checksum field value of 0x0000 indicates that the sender did not compute a checksum. If the sender did compute a checksum and the receiver detects a checksum error, the UDP datagram is silently discarded.


## IP Fragmentation
link-layer framing normally imposes an upper limit on the maximum size of a frame that can be transmitted. To keep the IP datagram
abstraction consistent and isolated from link-layer details, IP employs fragmentation and reassembly. IP compares the outgoing interface’s MTU with the datagram size and performs fragmentation if the datagram is too large.

When an IP datagram is fragmented, it is not reassembled until it reaches its final destination. 

An application using UDP may need to worry about the size of the resulting IP datagram it creates if it wishes to avoid IP-layer fragmentation.

If the size of the resulting datagram exceeds the link's MTU, the IP datagram is split
across multiple IP packets, which can lead to performance issues because if any fragment is lost, the entire datagram is lost.

# Maximum UDP Datagram Size
The maximum size of an IPv4 datagram is 65,535 bytes, imposed by the 16-bit Total Length field in the IPv4 header. With an optionless
IPv4 header of 20 bytes and a UDP header of 8 bytes, this leaves a maximum of 65,507 bytes of user data in a UDP datagram. For IPv6, the 16-bit Payload Length field permits an effective UDP payload of 65,527 bytes (8 of the 65,535 IPv6 payload bytes are used for the UDP header), assuming jumbograms are not being used.


# Restricting Local IP Addresses
Most UDP servers wildcard their local IP address when they bind a UDP endpoint. This means that an incoming UDP datagram destined for the server’s port is accepted on any local IP address (any IP address in use on the local machine, including the local loopback address).

# Restricting Foreign IP Address
The foreign IP address and foreign port number are shown as 0.0.0.0:*, meaning that the endpoint will accept an incoming UDP datagram from any IPv4 address and any port number.

# Lack of Flow and Congestion Control
Most UDP servers are iterative servers. This means that a single server thread (or process) handles all the client requests on a single UDP port (e.g., the server’s well-known port).

It is possible, however, for this queue to overflow, causing the UDP implementation to discard incoming datagrams. Another concern arises from the fact that queues are also present in the IP routers between the sender and the receiver—in the middle of the network. When these queues become full, traffic may be discarded in a fashion similar to that of the UDP input queue.

# Broadcast
Generally speaking, applications using  broadcast use the UDP protocol (or ICMPv4
protocol) and invoke an ordinary set of API calls to send traffic. 


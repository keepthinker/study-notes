# Introduction
In computing, load balancing improves the distribution of workloads across multiple computing resources, such as computers, a computer cluster, network links, central processing units.


## Client-side random load balancing
A approach to load balancing is to deliver a list of server IPs to the client, and then to have client randomly select the IP from the list on each connection.

## Server-side load balancers
For Internet services, a server-side load balancer is usually a software program that is listening on the port where external clients connect to access services. The load balancer forwards requests to one of the "backend" servers, which usually replies to the load balancer. This allows the load balancer to reply to the client without the client ever knowing about the internal separation of functions. It also prevents clients from contacting back-end servers directly, which may have security benefits by hiding the structure of the internal network and preventing attacks on the kernel's network stack or unrelated services running on other ports.

### Scheduling algorithms
Numerous scheduling algorithms, also called load-balancing methods, are used by load balancers to determine which back-end server to send a request to.[6] Simple algorithms include random choice, round robin, or least connections.


## LVS
### LVS-NAT（Network Address Translation）
First consider the following figure,

![image](http://www.linuxvirtualserver.org/VS-NAT.gif)

When a user accesses the service provided by the server cluster, the request packet destined for **virtual IP address (the external IP address for the load balancer)** arrives at the load balancer. The load balancer examines the packet's destination address and port number. If they are matched for a virtual server service according to the virtual server rule table, a real server is chosen from the cluster by a scheduling algorithm, and the connection is **added into the hash table** which record the established connection. Then, **the destination address and the port of the packet are rewritten to those of the chosen server**, and the packet is forwarded to the server. When the incoming packet belongs to this connection and the chosen server can be found in the hash table, the packet will be rewritten and forwarded to the chosen server. When the reply packets come back, the load balancer(**default route**) **rewrites the source address and port of the packets to those of the virtual service**. After the connection terminates or timeouts, the connection record will be removed in the hash table.

Confused? Let me give an example to make it clear. In the example, computers are configured as follows:
![image](LVS-NAT.gif)
Note real servers can run any OS that supports TCP/IP, **the default route of real servers must be the virtual server** (**172.16.0.1** in this example). The ipfwadm utility is used to make the virtual server accept packets from real servers. In the example above, the command is as follows:

```
    echo 1 > /proc/sys/net/ipv4/ip_forward
    ipfwadm -F -a m -S 172.16.0.0/24 -D 0.0.0.0/0
```

The following figure illustrates the rules specified in the Linux box with virtual server support.

Protocol  |  Virtual IP Address |  Port  |  Real IP Address	|  Port |  Weight
TCP	      |  202.103.106.5	    |  80    |   172.16.0.2	    |  80	|     1
          |  172.16.0.3	        |  8000  |                  |       |     2
TCP	      |  202.103.106.5	    |  21    |   172.16.0.3	    |  21	|     1

All traffic destined for IP address 202.103.106.5 Port 80 is load-balanced over real IP address 172.16.0.2 Port 80 and 172.16.0.3 Port 8000. Traffic destined for IP address 202.103.106.5 Port 21 is port-forwarded to real IP address 172.16.0.3 Port 21.

Packet rewriting works as follows.

The incoming packet for web service would has source and destination addresses as:

SOURCE	202.100.1.2:3456	DEST	202.103.106.5:80
The load balancer will choose a real server, e.g. 172.16.0.3:8000. The packet would be rewritten and forwarded to the server as:

SOURCE	202.100.1.2:3456	DEST	172.16.0.3:8000
Replies get back to the load balancer as:

SOURCE	172.16.0.3:8000	DEST	202.100.1.2:3456
The packets would be written back to the virtual server address and returned to the client as:

SOURCE	202.103.106.5:80	DEST	202.100.1.2:3456


客户通过Virtual IP Address（虚拟服务的IP地址）访问网络服务时，请求报文到达调度器，调度器根据连接调度算法从一组真实服务器中选出一台服务器，将**报文的目标地址Virtual IP Address改写成选定服务器的地址，报文的目标端口改写成选定服务器的相应端口**，最后将修改后的报文发送给选出的服务器。同时，调度器在连接Hash表中记录这个连接，当这个连接的下一个报文到达时，从连接Hash表中可以得到原选定服务器的地址和端口，进行同样的改写操作，并将报文传给原选定的服务器。当来自真实服务器的响应报文经过调度器时，调度器将报文的源地址和源端口改为Virtual IP Address和相应的端口，再把报文发给用户。

#### 概要流程
假设客户端请求报文，其中源地址为ip1:p1，目标地址为ip2:p2，经过公网路由到达LVS外网地址，也就是ip2:p2，在LVS-NAT模式下，LVS将客户端请求报文的目标IP和端口改成LVS下游的某个服务节点ip3:p3。因为**下游服务结点的默认路由指向的是LVS地址**，这个服务节点接收到客户端请求并处理完该次请求后，将回复报文发送给LVS，LVS再将回复保报文的源地址由原来的ip3:p3改为ip2:p2回复给客户端，在整个过程中，客户端端地址ip1:p1并未改变。

##### 地址转换流程

**client** --- <ip1:p1, ip2:p2> ---> **LVS**(ip2:p2) --- <ip1:p1, ip3:p3> ---> **RealServer**(ip3:p3) ---  <ip3:p3, ip1:p1>  ---> **LVS**(ip2:p2) --- <ip2:p2, ip1:p1> ---> **client**



### LVS-Tunnel
First, let's look at the figure of virtual server via **IP tunneling**. The most different thing of virtual server via IP tunneling to that of virtual server via NAT is that the load balancer sends requests to real servers through IP tunnel in the former, and the load balancer sends request to real servers via network address translation in the latter.

![image](VS-IPTunneling.gif)

When a user accesses a virtual service provided by the server cluster, a packet destined for virtual IP address (the IP address for the virtual server) arrives. The load balancer examines the packet's destination address and port. If they are matched for the virtual service, a real server is chosen from the cluster according to a connection scheduling algorithm, and the connection is added into the hash table which records connections. Then, the load balancer encapsulates the packet within an IP datagram and forwards it to the chosen server. When an incoming packet belongs to this connection and the chosen server can be found in the hash table, the packet will be again encapsulated and forwarded to that server. When the server receives the encapsulated packet, it decapsulates the packet and processes the request, finally **return the result directly to the user** according to its own routing table. After a connection terminates or timeouts, the connection record will be removed from the hash table. The workflow is illustrated in the following figure.

![image](VS-TUN-flow.jpg)

Note that real servers can have any real IP address in any network, they **can be geographically distributed**, but they must support IP encapsulation protocol. Their tunnel devices are all configured up so that the systems can decapsulate the received encapsulation packets properly, and **the <Virtual IP Address> must be configured on non-arp devices or any alias of non-arp devices, or the system can be configured to redirect packets for <Virtual IP Address> to a local socket.** See the arp problem page for more information.

Finally, when an encapsulated packet arrives, the real server decapsulates it and finds that **the packet is destined for <Virtual IP Address>**, it says, "Oh, it is for me, so I do it.", it processes the request and returns the result directly to the user in the end.

它的连接调度和管理与VS/NAT中的一样，只是它的报文转发方法不同。调度器根据各个服务器的负载情况，动态地选择一台服务器，将**请求报文封装在另一个IP报文中**，再将封装后的IP报文转发给选出的服务器；服务器收到报文后，先将报文解封获得原来目标地址为VIP的报文，服务器发现VIP地址被配置在本地的IP隧道设备上，所以就处理这个请求，然后根据路由表将响应报文直接返回给客户。

#### 概要流程
假设客户端请求报文，其中源地址为ip1:p1，目标地址为ip2:p2，经过公网路由到达LVS外网地址，也就是ip2:p2，LVS将该IP报文分装在另一个报文中，此时IP报文的目标地址为LVS下游的某个节点的IP，节点接收到LVS转发的报文解封后发现VIP地址被配置在本地的IP隧道设备上，就处理该请求，处理完毕后，将报文直接返回给客户端，当然此时源地址和目标地址为ip2:p2和ip1:ip1，此时不一定经过LVS服务器。

##### 地址转换流程

**client** --- <ip1:p1, ip2:p2> ---> **LVS**(ip2:p2) --- (ip3<ip1:p1, ip2:p2>) ---> **RealServer**(ip3) ---  <ip2:p2, ip1:p1>  ---> **client**

RealServer节点服务器需要在**本地回环接口配置VIP**，和LVS一样的VIP地址。

#### LVS-DR(Direct Routing)
This request dispatching approach is similar to the one implemented in IBM's NetDispatcher. **The virtual IP address is shared by real servers and the load balancer**. The load balancer has an interface configured with the virtual IP address too, which is used to accept request packets, and it directly route the packets to the chosen servers. **All the real servers have their non-arp alias interface configured with the virtual IP address or redirect packets destined for the virtual IP address to a local socket, so that the real servers can process the packets locally.** The load balancer and the real servers must have one of their interfaces physically linked by a HUB/Switch. 

![image](VS-DRouting.gif)

When a user accesses a virtual service provided by the server cluster, the packet destined for virtual IP address (the IP address for the virtual server) arrives. The load balancer(LinuxDirector) examines the packet's destination address and port. If they are matched for a virtual service, a real server is chosen from the cluster by a scheduling algorithm, and **the connection is added into the hash table which records connections**. Then, the load balancer directly forwards it to the chosen server. When the incoming packet belongs to this connection and the chosen server can be found in the hash table, the packet will be again directly routed to the server. When the server receives the forwarded packet, the server finds that the packet is for the address on its alias interface or for a local socket, so it processes the request and return the result directly to the user finally. After a connection terminates or timeouts, the connection record will be removed from the hash table.

The direct routing workflow is illustrated in the following figure:
![image](VS-DR-flow.jpg)

The load balancer **simply changes the MAC address of the data frame** to that of the chosen server and restransmits it on the LAN. This is the reason that the load balancer and each server must be directly connected to one another by a single uninterrupted segment of a LAN. If you meet some arp problem of the cluster, see the arp problem page for more information. 

**调度器和服务器组都必须在物理上有一个网卡通过不分段的局域网相连，即通过交换机或者高速的HUB相连，中间没有隔有路由器**。VIP地址为调度器和服务器组共享，调度器配置的VIP地址是对外可见的，用于接收虚拟服务的请求报文；所有的服务器把VIP地址配置在各自的Non-ARP网络设备上，它对外面是不可见的，只是用于处理目标地址为VIP的网络请求。

它的连接调度和管理与VS/NAT和VS/TUN中的一样，它的报文转发方法又有不同，将报文直接路由给目标服务器。在VS/DR中，调度器根据各个服务器的负载情况，动态地选择一台服务器，**不修改也不封装IP报文**，而是将**数据帧的MAC地址改为选出服务器的MAC地址**，再将修改后的数据帧在与服务器组的局域网上发送。

##### 地址转换流程

**client** --- <ip1:p1, ip2:p2> ---> **LVS**(m1, ip2:p2) --- ((m1, m2)<ip1:p1, ip2:p2>) ---> **RealServer**(m2, ip3) ---  <ip2:p2, ip1:p1>  ---> **client**

RealServer节点服务器需要在**本地回环接口配置VIP**，和LVS一样的VIP地址。

### LVS-fullnat
1. 客户端将访问vip报文发送给LVS服务器；
2. LVS服务器将请求报文的目的地址修改为后端真实服务器(DNAT)，源地址改为自己的ip地址(SNAT)，发送给后端真实服务器；
3. 后端服务器在处理完之后要将响应的报文返回给lvs；
4. LVS将返回的数据包源地址改为自己(SNAT)，目的地址改为客户端(DNAT)，发送给客户端。


# Haproxy
HAProxy is a free, very fast and reliable solution offering high availability, load balancing, and proxying for **TCP** and **HTTP-based** applications. It is particularly suited for very high traffic web sites and powers quite a number of the world's most visited ones. Over the years it has become the de-facto standard opensource load balancer, is now shipped with most mainstream Linux distributions, and is often deployed by default in cloud platforms. 

# NGINX
nginx的负载均衡策略可以划分为两大类：内置策略和扩展策略。内置策略包含加权轮询和ip hash。扩展策略有很多，如fair、通用hash、consistent hash等，默认不编译进nginx内核。

# DNS（Domain Name System）
DNS负载均衡最早的负载均衡技术是通过DNS来实现的，在DNS中为 名字，因而查询这个名字的客户机将得到其中一个地址，从而使得不同的客户访问不同的服务器，达到负载均衡的目的。DNS负载均衡是一种简单而有效的方法，但是它不能区分服务器的差异，也不能反映服务器的当前运行状态。


# F5

F5's BIG-IP is a family of products covering **software and hardware** designed around application availability, access control, and security solutions.  That's right, the BIG-IP name is interchangeable between F5's software and hardware application delivery controller and security products.  This is different from BIG-IQ, a suite of management and orchestration tools, and F5 Silverline, F5's SaaS platform.  When people refer to BIG-IP this can mean a single software module in BIG-IP's software family or it could mean a hardware chassis sitting in your datacenter.  This can sometimes cause a lot of confusion when people say they have question about "BIG-IP" but we'll break it down here to reduce the confusion.


## More than load balancer
BIG-IP Local Traffic Manager enables you to control network traffic, selecting the right destination based on server performance, security, and availability.

## Read the datasheet
Full proxy means full power
Because BIG-IP LTM is a full proxy, you can inspect, manage, and report on application traffic entering and exiting your network. From basic load balancing to complex traffic management decisions based on client, server, or application status, BIG-IP LTM gives you granular control over app traffic.

## Blazing fast SSL
The SSL performance of BIG-IP LTM lets you cost-effectively protect the end-to-end user experience by encrypting everything from the client to the server. It also scales on-demand and absorbs potentially crippling DDoS attacks. BIG-IP LTM includes levels of inspection necessary to block bad traffic and allow good traffic to pass through.

## TCP Optimization
The highly optimized TCP/IP stack, TCP Express, combines TCP/IP techniques and improvements in the latest RFCs with extensions to minimize the effect of congestion and packet loss and recovery. Independent testing tools and customer experiences show TCP Express delivers up to a 2x performance gain for users and a 4x increase in bandwidth efficiency.    

## Performance optimization
BIG-IP LTM can optimize the speed and reliability of your applications via both network and application layers. Using real-time protocol and traffic management decisions based on application and server conditions, extensive connection management, and TCP and content offloading, BIG-IP LTM dramatically improves page load times. 

## Programmability
BIG-IP LTM is programmable, so you can take the visibility and control it provides and immediately act on it using iRules, F5’s event-driven scripting language. From defeating zero-day attacks to cloning specific app requests or dealing with custom application protocols, iRules let you adapt to application delivery challenges across any environment.

## Scale and Speed
With BIG-IP LTM, you get a sophisticated, enterprise-class load balancer. You also get granular layer 7 control, SSL offloading and acceleration capabilities, and ScaleN technology that delivers on-demand scaling.


[负载均衡获得真实源IP的6种方法](https://mp.weixin.qq.com/s?__biz=MzI1OTU2MDA4NQ==&mid=2247487922&idx=1&sn=342e913a12e2bb023da2db6c322a647f&chksm=ea765648dd01df5e35179b8978f3456e29cfb7af878e2ff8df5e3e739531d5f90c757d98ca7e&mpshare=1&scene=1&srcid=1121zvim3WNSeehtHhpLJKLE#rd)

[IP负载均衡技术](http://zh.linuxvirtualserver.org/node/25)

[Load_balancing_(computing)](https://en.wikipedia.org/wiki/Load_balancing_(computing))

[LVS-四种工作模式( dr tunnel nat fullnat )原理简介](https://blog.csdn.net/sr_1114/article/details/80256626)

[LVS DR/NAT/FULLNAT/TUNNEL模式介绍](https://blog.csdn.net/wjxydzxxyy/article/details/50969432)

[Virtual Server](http://www.linuxvirtualserver.org/)

[F5 Official Website](https://devcentral.f5.com/)

[负载均衡总结（四层负载与七层负载的区别）](https://www.jianshu.com/p/9826d866080a)
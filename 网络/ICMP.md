# **I**nternet **C**ontrol **M**essage **P**rotocol（互联网控制消息协议）

它用于[网际协议](https://zh.wikipedia.org/wiki/网际协议)（IP）中发送控制消息，提供可能发生在通信环境中的各种问题反馈。通过这些信息，使管理者可以对所发生的问题作出诊断，然后采取适当的措施解决。

CMP [[1\]](https://zh.wikipedia.org/wiki/互联网控制消息协议#cite_note-rfc792-1)依靠IP来完成它的任务，它是IP的主要部分。它与传输协议（如[TCP](https://zh.wikipedia.org/wiki/传输控制协议)和[UDP](https://zh.wikipedia.org/wiki/用户数据报协议)）显著不同：它一般不用于在两点间传输数据。它通常不由网络程序直接使用，除了 [ping](https://zh.wikipedia.org/wiki/Ping) 和 [traceroute](https://zh.wikipedia.org/wiki/Traceroute) 这两个特别的例子。 [IPv4](https://zh.wikipedia.org/wiki/IPv4)中的ICMP被称作ICMPv4，[IPv6](https://zh.wikipedia.org/wiki/IPv6)中的ICMP则被称作[ICMPv6](https://zh.wikipedia.org/wiki/ICMPv6)。

## 技术细节

虽然ICMP是包含在IP数据包中的，但是对ICMP消息通常会特殊处理，会和一般IP数据包的处理不同，而不是作为IP的一个子协议来处理。在很多时候，需要去查看ICMP消息的内容，然后发送适当的错误消息到那个原来产生IP数据包的程序，即那个导致ICMP消息被发送的IP数据包。

很多常用的工具是基于ICMP消息的。[traceroute](https://zh.wikipedia.org/wiki/Traceroute) 是通过发送包含有特殊的TTL的包，然后接收ICMP超时消息和目标不可达消息来实现的。 [ping](https://zh.wikipedia.org/wiki/Ping) 则是用ICMP的"Echo request"（类别代码：8）和"Echo reply"（类别代码：0）消息来实现的。

## ICMP报文结构

ICMP报头从IP报头的第160位开始（IP首部20字节）（除非使用了IP报头的可选部分）。

然后协议为：Type(8), Code(8), checksum(16), RestOfHeader(32)

|                                                      |      |                                                              |                                                              |      |      |
| :--------------------------------------------------: | :--: | :----------------------------------------------------------: | :----------------------------------------------------------: | :--: | :--: |
|                         类型                         | 代码 |                             状态                             |                             描述                             | 查询 | 差错 |
| 0 - [Echo Reply](https://zh.wikipedia.org/wiki/Ping) |  0   |                                                              | echo响应 (被程序[ping](https://zh.wikipedia.org/wiki/Ping)使用） |  ●   |      |
|                       1 and 2                        |      |                            未分配                            |                             保留                             |      |  ●   |
|                    3 - 目的不可达                    |  0   |                                                              |                        目标网络不可达                        |      |  ●   |
|                          1                           |      |                        目标主机不可达                        |                                                              |  ●   |      |
|                          2                           |      |                        目标协议不可达                        |                                                              |  ●   |      |
|                          3                           |      |                        目标端口不可达                        |                                                              |  ●   |      |
|                          4                           |      | 要求分段并设置[DF flag](https://zh.wikipedia.org/wiki/IPv4#报文结构)标志 |                                                              |  ●   |      |
|                          5                           |      |                          源路由失败                          |                                                              |  ●   |      |
|                          6                           |      |                        未知的目标网络                        |                                                              |  ●   |      |
|                          7                           |      |                        未知的目标主机                        |                                                              |  ●   |      |
|                          8                           |      |                    源主机隔离（作废不用）                    |                                                              |  ●   |      |
|                          9                           |      |                        禁止访问的网络                        |                                                              |  ●   |      |
|                          10                          |      |                        禁止访问的主机                        |                                                              |  ●   |      |
|                          11                          |      |                    对特定的TOS 网络不可达                    |                                                              |  ●   |      |
|                          12                          |      |                    对特定的TOS 主机不可达                    |                                                              |  ●   |      |
|                          13                          |      |                   由于过滤 网络流量被禁止                    |                                                              |  ●   |      |
|                          14                          |      |                           主机越权                           |                                                              |  ●   |      |
|                          15                          |      |                        优先权终止生效                        |                                                              |  ●   |      |
|                     4 - 源端关闭                     |  0   |                             弃用                             |                     源端关闭（拥塞控制）                     |      |  ●   |
|                      5 - 重定向                      |  0   |                                                              |                          重定向网络                          |      |  ●   |
|                          1                           |      |                          重定向主机                          |                                                              |  ●   |      |
|                          2                           |      |                     基于TOS 的网络重定向                     |                                                              |  ●   |      |
|                          3                           |      |                     基于TOS 的主机重定向                     |                                                              |  ●   |      |
|                          6                           |      |                             弃用                             |                         备用主机地址                         |      |      |
|                          7                           |      |                            未分配                            |                             保留                             |      |      |
|  8 - [请求回显](https://zh.wikipedia.org/wiki/Ping)  |  0   |                                                              |                           Echo请求                           |  ●   |      |
|                    9 - 路由器通告                    |  0   |                                                              |                           路由通告                           |  ●   |      |
|                   10 - 路由器请求                    |  0   |                                                              |                    路由器的发现/选择/请求                    |  ●   |      |
|                    11 - ICMP 超时                    |  0   |                                                              |                           TTL 超时                           |      |  ●   |
|                          1                           |      |                         分片重组超时                         |                                                              |  ●   |      |
|              12 - 参数问题：错误IP头部               |  0   |                                                              |                      IP 报首部参数错误                       |      |  ●   |
|                          1                           |      |                         丢失必要选项                         |                                                              |  ●   |      |
|                          2                           |      |                         不支持的长度                         |                                                              |      |      |
|                   13 - 时间戳请求                    |  0   |                                                              |                          时间戳请求                          |  ●   |      |
|                   14 - 时间戳应答                    |  0   |                                                              |                          时间戳应答                          |  ●   |      |
|                    15 - 信息请求                     |  0   |                             弃用                             |                           信息请求                           |  ●   |      |
|                    16 - 信息应答                     |  0   |                             弃用                             |                           信息应答                           |  ●   |      |
|                  17 - 地址掩码请求                   |  0   |                             弃用                             |                         地址掩码请求                         |  ●   |      |
|                  18 - 地址掩码应答                   |  0   |                             弃用                             |                         地址掩码应答                         |  ●   |      |
|                          19                          |      |                             保留                             |                        因安全原因保留                        |      |      |
|                       20 至 29                       |      |                             保留                             |             *Reserved* for robustness experiment             |      |      |
|                   30 - Traceroute                    |  0   |                             弃用                             |                           信息请求                           |      |      |
|                          31                          |      |                             弃用                             |                        数据报转换出错                        |      |      |
|                          32                          |      |                             弃用                             |                        手机网络重定向                        |      |      |
|                          33                          |      |                             弃用                             | [Where-Are-You](https://zh.wikipedia.org/w/index.php?title=Where-Are-You&action=edit&redlink=1)（originally meant for [IPv6](https://zh.wikipedia.org/wiki/IPv6)） |      |      |
|                          34                          |      |                             弃用                             | [Here-I-Am](https://zh.wikipedia.org/w/index.php?title=Where-Are-You&action=edit&redlink=1)（originally meant for IPv6） |      |      |
|                          35                          |      |                             弃用                             |                 Mobile Registration Request                  |      |      |
|                          36                          |      |                             弃用                             |                  Mobile Registration Reply                   |      |      |
|                          37                          |      |                             弃用                             |                     Domain Name Request                      |      |      |
|                          38                          |      |                             弃用                             |                      Domain Name Reply                       |      |      |
|                          39                          |      |                             弃用                             | SKIP Algorithm Discovery Protocol, [Simple Key-Management for Internet Protocol](https://zh.wikipedia.org/w/index.php?title=Simple_Key-Management_for_Internet_Protocol&action=edit&redlink=1) |      |      |
|                          40                          |      |                                                              | [Photuris](https://zh.wikipedia.org/w/index.php?title=Photuris_(protocol)&action=edit&redlink=1), Security failures |      |      |
|                          41                          |      |                           实验性的                           | ICMP for experimental mobility protocols such as [Seamoby](https://zh.wikipedia.org/w/index.php?title=Seamoby&action=edit&redlink=1) [RFC4065] |      |      |
|                      42 到 255                       |      |                             保留                             |                             保留                             |      |      |
|                         235                          |      |                           实验性的                           | RFC3692（ [RFC 4727](https://tools.ietf.org/html/rfc4727)）  |      |      |
|                         254                          |      |                           实验性的                           | RFC3692（ [RFC 4727](https://tools.ietf.org/html/rfc4727)）  |      |      |
|                         255                          |      |                             保留                             |                             保留                             |      |      |



## 参考文献

[互联网控制消息协议](https://zh.wikipedia.org/wiki/%E4%BA%92%E8%81%94%E7%BD%91%E6%8E%A7%E5%88%B6%E6%B6%88%E6%81%AF%E5%8D%8F%E8%AE%AE)
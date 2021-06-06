# 工具
![image](https://note.youdao.com/yws/api/personal/file/9015459827D045DA88073161E2F5D33F?method=getImage&version=5794&cstk=dipEa6If)
# vmstat
## FIELD DESCRIPTION FOR VM MODE
### Procs
- r: The number of processes waiting for run time.
- b: The number of processes in uninterruptible sleep.
### Memory
- swpd: the amount of virtual memory used.
- free: the amount of idle memory.
- buff: the amount of memory used as buffers.
- cache: the amount of memory used as cache.
- inact: the amount of inactive memory. (-a option)
- active: the amount of active memory. (-a option)

### Swap
- si: Amount of memory swapped in from disk (/s).
- so: Amount of memory swapped to disk (/s).

### IO
- bi: Blocks received from a block device (blocks/s).
- bo: Blocks sent to a block device (blocks/s).

### System
- in: The number of interrupts per second, including the clock.
- cs: The number of context switches per second.

### CPU
These are percentages of total CPU time.
- us: Time spent running non-kernel code. (user time, including nice time)
- sy: Time spent running kernel code. (system time)
- id: Time spent idle. Prior to Linux 2.5.41, this includes IO-wait time.
- wa: Time spent waiting for IO. Prior to Linux 2.5.41, included in idle.
- st: Time stolen from a virtual machine. Prior to Linux 2.6.11, unknown.


# iostat
```
iostat 1

avg-cpu: %user %nice %system %iowait  %steal   %idle (refer to #top/##cpu)

Device:  tps kB_read/s kB_wrtn/s kB_read kB_wrtn
```

**tps**: iops. Indicate the number of transfers per second that were issued to the device. A transfer is an I/O request to the device.  Multiple logical requests can be combined into a single I/O request to the device. A transfer is of indeterminate size.

**Blk_read/s (kB_read/s, MB_read/s)**: Indicate  the amount of data read from the device expressed in a number of blocks (kilobytes, megabytes) per second. Blocks are equivalent to sectors and therefore have a size of 512 bytes.

**Blk_wrtn/s (kB_wrtn/s, MB_wrtn/s)**: Indicate the amount of data written to the device expressed in a number of blocks (kilobytes, megabytes) per second.

**Blk_read (kB_read, MB_read)**:  The total number of blocks (kilobytes, megabytes) read.

**Blk_wrtn (kB_wrtn, MB_wrtn)**: The total number of blocks (kilobytes, megabytes) written.


# top
## cpu
- us, user    : time running un-niced user processes
- sy, system  : time running kernel processes
- ni, nice    : time running niced user processes
- id, idle    : time spent in the kernel idle handler
- wa, IO-wait : time waiting for I/O completion
- hi : time spent servicing hardware interrupts
- si : time spent servicing software interrupts
- st : time stolen from this vm by the hypervisor

### 翻译
- us 用户空间占用CPU百分比
- sy 内核空间占用CPU百分比
- ni 用户进程空间内改变过优先级的进程占用CPU百分比
- id 空闲CPU百分比
- wa 等待输入输出的CPU时间百分比
- hi 硬件中断
- si 软件中断 
- st: 实时

## field
**PID**: 进程 ID 

**USER**: 进程所有者的用户名 

**PR**: 任务优先级 

**NI**: nice 值。A negative nice value means higher priority, whereas a positive nice value means lower priority. Zero in this field simply means priority will not be adjusted in determining a task's dispatch-ability.

**VIRT**: The  total  amount of virtual memory used by the task.  It includes all code, data and shared libraries plus pages that have been  swapped out and pages that have been mapped but not used. VIRT=SWAP+RES 

**RES**: 进程使用的、未被换出的物理内存大小，单位：kb。RES=CODE+DATA 

**SHR**: 共享内存大小，单位：kb 

**S**: 进程状态。 
- D= 不可中断的睡眠状态
- R= 运行
- S= 睡眠
- T= 跟踪 / 停止
- Z=僵尸进程

**%CPU**: The task's share of the elapsed CPU time since the last screen update, expressed as a percentage of total CPU time.

**%MEM%**: A task's currently used share of available phisical memory.

**TIME+**: 进程使用的 CPU 时间总计，精确到 1/100 秒 

**COMMAND**": 命令名 / 命令行

## 查看进程线程情况
top -H -p $pid

# free
Display amount of free and used memory in the system



# uptime
Gives a one line display of the following information.  The current time, how long the system has been running, how many users are currently logged on, and the system **load averages** for the past **1, 5, and 15** minutes.

# pidstat
```
//-u: cpu情况
//-r: 内存情况
//每2秒统计一次
pidstat -p $pid -u -r 2
```

# ss & netstat
ss比netstat快的主要原因是，netstat是遍历/proc下面每个PID目录，ss直接读/proc/net下面的统计信息。所以ss执行的时候消耗资源以及消耗的时间都比netstat少很多。

当服务器的socket连接数量非常大时（如上万个），无论是使用netstat命令还是直接cat /proc/net/tcp执行速度都会很慢，相比之下ss可以节省很多时间
```
ss -anpt

netstat -anpt

//总体统计
ss -s
```
## ss -s源码
```
printf("TCP: %d (estab %d, closed %d, orphaned %d, synrecv %d, timewait %d/%d), ports %d\n",
s.tcp_total + slabstat.tcp_syns + s.tcp_tws,
sn.tcp_estab,
s.tcp_total - (s.tcp4_hashed+s.tcp6_hashed-s.tcp_tws),
s.tcp_orphans,
slabstat.tcp_syns,
s.tcp_tws, slabstat.tcp_tws,
slabstat.tcp_ports);

```

[ss源代码github](https://github.com/sivasankariit/iproute2/blob/master/misc/ss.c)

# tcpdump
```
//interface网络接口名
tcpdump -i $interface 
```


# ifconfig
<pre>
eth0      Link encap:Ethernet  HWaddr dc:0e:a1:7b:f6:97  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
          Interrupt:16
</pre>
- eth0：网络接口名称。
- Link encap：链路层打包
- HWaddr：物理地址也就是MAC地址
- UP：网络设备/网卡的开启状态
- MTU：Maximum Transmission Unit，当前eth0接口的最大传输单元。
- Metric：将接口度量值设置为整数number。度量值表示在这个路径上发送一个分组的成本。目前内核中还没有使用路由成本，但将来会。
- RX & TX：RX（接收分组数）和TX（传送分组数）这两行显示出了接收、传送分组的数目，以及分组出错数、丢失分组数（一个可能原因是内存较少）和超限数（通常在接收器接收数据的速度快于核心的处理速度的时候发生）。
- Interrupt：Interrupt numbers are assigned to hardware devices by the kernel. They are used to multiplex the few interrupt channels that the CPU has between many hardware devices. You can see all interrupt numbers being used by executing cat /proc/interrupts

# ip
```
//查看出口路由地址
ip route
//增加默认路由
ip route add default gw 20.0.0.1
//显示网络接口eth0信息
ip addr show eth0
//查看网络的统计信息，比如所有网卡上传输的字节数和报文数，错误或丢弃的报文数等
ip -s link
```

# dmesg
## 简介
dmesg命令用于打印Linux系统开机启动信息，kernel会将开机信息存储在ring buffer中。您若是开机时来不及查看信息，可利用dmesg来查看（print or control the kernel ring buffer）。开机信息亦保存在/var/log/dmesg的文件里。某些硬件设备（比如七号信令卡、语音卡之类）在安装的时候，通常会安装驱动程序（内核模块），会打印一些信息，就可以通过dmesg命令来查看。





# lsof


1.列出所有打开的文件:
```
lsof
```

备注: 如果不加任何参数，就会打开所有被打开的文件，建议加上一下参数来具体定位

2. 查看谁正在使用某个文件
```
lsof /filepath/file
```
3.递归查看某个目录的文件信息
```
lsof +D /filepath/filepath2/
```
备注: 使用了+D，对应目录下的所有子目录和文件都会被列出

4. 比使用+D选项，遍历查看某个目录的所有文件信息 的方法
```
lsof | grep ‘/filepath/filepath2/’
```
5. 列出某个用户打开的文件信息
```
lsof -u username
```
备注: -u 选项，u其实是user的缩写

6. 列出某个程序所打开的文件信息
```
lsof -c mysql
```
备注: -c 选项将会列出所有以mysql开头的程序的文件，其实你也可以写成 lsof | grep mysql, 但是第一种方法明显比第二种方法要少打几个字符了

7. 列出多个程序多打开的文件信息
```
lsof -c mysql -c apache
```
8. 列出某个用户以及某个程序所打开的文件信息
```
lsof -u test -c mysql
```
9. 列出除了某个用户外的被打开的文件信息
```
lsof -u ^root
```
备注：^这个符号在用户名之前，将会把是root用户打开的进程不让显示

10. 通过某个进程号显示该进行打开的文件
```
lsof -p 1
```
11. 列出多个进程号对应的文件信息
```
lsof -p 123,456,789
```
12. 列出除了某个进程号，其他进程号所打开的文件信息
```
lsof -p ^1
```
13 . 列出所有的网络连接
```
lsof -i
```
14. 列出所有tcp 网络连接信息
```
lsof -i tcp
```
15. 列出所有udp网络连接信息
```
lsof -i udp
```
16. 列出谁在使用某个端口
```
lsof -i :3306
```
17. 列出谁在使用某个特定的udp端口
```
lsof -i udp:55
```
特定的tcp端口
```
lsof -i tcp:80
```
18. 列出某个用户的所有活跃的网络端口
```
lsof -a -u test -i
```
19. 列出所有网络文件系统
```
lsof -N
```
20. 域名socket文件
```
lsof -u
```
21. 某个用户组所打开的文件信息
```
lsof -g 5555
```
22. 根据文件描述列出对应的文件信息
```
lsof -d description(like 2)
```
23. 根据文件描述范围列出文件信息
```
lsof -d 2-3
```

## 参考文献
[linux命令 — lsof 查看进程打开那些文件 或者 查看文件给那个进程使用](https://blog.csdn.net/kozazyh/article/details/5495532 )


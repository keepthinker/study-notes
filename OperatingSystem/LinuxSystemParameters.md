# 目录
1. sysctl
2. ulimit
3. 网卡参数



# sysctl -a
The parameters available are those listed under /proc/sys/

website: 

https://www.kernel.org/doc/Documentation/sysctl/

https://linux.die.net/man/7


## abi
```
abi.vsyscall32 = 1
```
## debug
```
debug.exception-trace = 1
debug.kprobes-optimization = 1
```
## dev
```
dev.cdrom.autoclose = 1
dev.cdrom.autoeject = 0
dev.cdrom.check_media = 0
dev.cdrom.debug = 0
dev.cdrom.info = CD-ROM information, Id: cdrom.c 3.20 2003/12/17
dev.cdrom.info = 
dev.cdrom.info = drive name:		sr0
dev.cdrom.info = drive speed:		24
dev.cdrom.info = drive # of slots:	1
dev.cdrom.info = Can close tray:		1
dev.cdrom.info = Can open tray:		1
dev.cdrom.info = Can lock tray:		1
dev.cdrom.info = Can change speed:	1
dev.cdrom.info = Can select disk:	0
dev.cdrom.info = Can read multisession:	1
dev.cdrom.info = Can read MCN:		1
dev.cdrom.info = Reports media changed:	1
dev.cdrom.info = Can play audio:		1
dev.cdrom.info = Can write CD-R:		1
dev.cdrom.info = Can write CD-RW:	1
dev.cdrom.info = Can read DVD:		1
dev.cdrom.info = Can write DVD-R:	1
dev.cdrom.info = Can write DVD-RAM:	1
dev.cdrom.info = Can read MRW:		1
dev.cdrom.info = Can write MRW:		1
dev.cdrom.info = Can write RAM:		1
dev.cdrom.info = 
dev.cdrom.info = 
dev.cdrom.lock = 0
dev.hpet.max-user-freq = 64
dev.mac_hid.mouse_button2_keycode = 97
dev.mac_hid.mouse_button3_keycode = 100
dev.mac_hid.mouse_button_emulation = 0
dev.parport.default.spintime = 500
dev.parport.default.timeslice = 200
dev.raid.speed_limit_max = 200000
dev.raid.speed_limit_min = 1000
dev.scsi.logging_level = 0
```

## fs

### file-max
The value in file-max denotes the maximum number of file-
handles that the Linux kernel will allocate. When you get lots
of error messages about running out of file handles, you might
want to increase this limit.

### file-nr
Historically, the kernel was able to allocate file handles
dynamically, but not to free them again. The three values in
file-nr denote **the number of allocated file handles**, **the number
of allocated but unused file handles**, and **the maximum number of
file handles**. Linux 2.6 always reports 0 as the number of free
file handles -- this is not an error, it just means that the
number of allocated file handles exactly matches the number of
used file handles.

### nr_open
This denotes the  **maximum number of file-handles a process can
allocate**. Default value is 1024*1024 (1048576) which should be
enough for most machines. Actual limit depends on RLIMIT_NOFILE
resource limit.

### inode-max, inode-nr & inode-state:

As with file handles, the kernel allocates the inode structures
dynamically, but can't free them yet.

The value in **inode-max** denotes **the maximum number of inode
handlers**. This value should be 3-4 times larger than the value
in file-max, since stdin, stdout and network sockets also
need an inode struct to handle them. When you regularly run
out of inodes, you need to increase this value.

#### inode-nr
The file 
**inode-nr**contains the first two items from
inode-state, so we'll skip to that file...

#### Inode-state
contains three actual numbers and four dummies.
The actual numbers are, in order of appearance, **nr_inodes**,
**nr_free_inodes** and **preshrink**.

##### **nr_inodes**
stands for the number of inodes the system has
allocated, this can be slightly more than inode-max because
Linux allocates them one pageful at a time.

##### **nr_free_inodes**
represents the number of free inodes (?) 

##### preshrink
is nonzero when the nr_inodes > inode-max and the
system needs to prune the inode list instead of allocating
more.


```
fs.aio-max-nr = 65536
fs.aio-nr = 5576
fs.dentry-state = 272034	256072	45	0	0	0
fs.dir-notify-enable = 1
fs.epoll.max_user_watches = 1658265
fs.file-max = 804694
fs.file-nr = 7943	0	804694
fs.inode-nr = 193177	420
fs.inode-state = 193177	420	0	0	0	0	0
fs.inotify.max_queued_events = 16384
fs.inotify.max_user_instances = 128
fs.inotify.max_user_watches = 8192
fs.lease-break-time = 45
fs.leases-enable = 1
fs.mqueue.msg_default = 10
fs.mqueue.msg_max = 10
fs.mqueue.msgsize_default = 8192
fs.mqueue.msgsize_max = 8192
fs.mqueue.queues_max = 256
fs.nr_open = 1048576
fs.overflowgid = 65534
fs.overflowuid = 65534
fs.pipe-max-size = 1048576
fs.pipe-user-pages-hard = 0
fs.pipe-user-pages-soft = 16384
fs.quota.allocated_dquots = 0
fs.quota.cache_hits = 0
fs.quota.drops = 0
fs.quota.free_dquots = 0
fs.quota.lookups = 0
fs.quota.reads = 0
fs.quota.syncs = 32
fs.quota.writes = 0
fs.suid_dumpable = 2
```
## kernel
```
kernel.acct = 4	2	30
kernel.acpi_video_flags = 0
kernel.auto_msgmni = 1
kernel.bootloader_type = 114
kernel.bootloader_version = 2
kernel.cap_last_cap = 37
kernel.compat-log = 1
kernel.core_pattern = |/usr/share/apport/apport %p %s %c %d %P
kernel.core_pipe_limit = 0
kernel.core_uses_pid = 0
kernel.ctrl-alt-del = 0
kernel.dmesg_restrict = 0
kernel.domainname = (none)
kernel.ftrace_dump_on_oops = 0
kernel.ftrace_enabled = 1
kernel.hostname = keepthinker-Lenovo-IdeaPad-Y470
kernel.hotplug = 
kernel.hung_task_check_count = 4194304
kernel.hung_task_panic = 0
kernel.hung_task_timeout_secs = 120
kernel.hung_task_warnings = 10
kernel.io_delay_type = 1
kernel.kexec_load_disabled = 0
kernel.keys.gc_delay = 300
kernel.keys.maxbytes = 20000
kernel.keys.maxkeys = 200
kernel.keys.persistent_keyring_expiry = 259200
kernel.keys.root_maxbytes = 20000
kernel.keys.root_maxkeys = 200
kernel.kptr_restrict = 1
kernel.kstack_depth_to_print = 12
kernel.max_lock_depth = 1024
kernel.modprobe = /sbin/modprobe
kernel.modules_disabled = 0
kernel.moksbstate_disabled = 0
kernel.msg_next_id = -1
kernel.msgmax = 8192
kernel.msgmnb = 16384
kernel.msgmni = 15851
kernel.ngroups_max = 65536
kernel.nmi_watchdog = 1
kernel.ns_last_pid = 27171
kernel.numa_balancing = 0
kernel.numa_balancing_scan_delay_ms = 1000
kernel.numa_balancing_scan_period_max_ms = 60000
kernel.numa_balancing_scan_period_min_ms = 1000
kernel.numa_balancing_scan_size_mb = 256
kernel.osrelease = 3.16.0-77-generic
kernel.ostype = Linux
kernel.overflowgid = 65534
kernel.overflowuid = 65534
kernel.panic = 0
kernel.panic_on_io_nmi = 0
kernel.panic_on_oops = 0
kernel.panic_on_unrecovered_nmi = 0
kernel.perf_cpu_time_max_percent = 25
kernel.perf_event_max_sample_rate = 50000
kernel.perf_event_mlock_kb = 516
kernel.perf_event_paranoid = 1
kernel.pid_max = 32768
kernel.poweroff_cmd = /sbin/poweroff
kernel.print-fatal-signals = 0
kernel.printk = 4	4	1	7
kernel.printk_delay = 0
kernel.printk_ratelimit = 5
kernel.printk_ratelimit_burst = 10
kernel.pty.max = 4096
kernel.pty.nr = 24
kernel.pty.reserve = 1024
kernel.random.boot_id = 3dec0f3c-8e71-45cb-bd23-759a6dd8a44a
kernel.random.entropy_avail = 857
kernel.random.poolsize = 4096
kernel.random.read_wakeup_threshold = 64
kernel.random.urandom_min_reseed_secs = 60
kernel.random.uuid = bbe21153-b70f-412f-b414-0a2207776edd
kernel.random.write_wakeup_threshold = 896
kernel.randomize_va_space = 2
kernel.real-root-dev = 0
kernel.sched_autogroup_enabled = 1
kernel.sched_cfs_bandwidth_slice_us = 5000
kernel.sched_child_runs_first = 0
kernel.sched_domain.cpu0.domain0.busy_factor = 32
kernel.sched_domain.cpu0.domain0.busy_idx = 0
kernel.sched_domain.cpu0.domain0.cache_nice_tries = 0
kernel.sched_domain.cpu0.domain0.flags = 687
kernel.sched_domain.cpu0.domain0.forkexec_idx = 0
kernel.sched_domain.cpu0.domain0.idle_idx = 0
kernel.sched_domain.cpu0.domain0.imbalance_pct = 110
kernel.sched_domain.cpu0.domain0.max_interval = 4
kernel.sched_domain.cpu0.domain0.max_newidle_lb_cost = 35080
kernel.sched_domain.cpu0.domain0.min_interval = 2
kernel.sched_domain.cpu0.domain0.name = SMT
kernel.sched_domain.cpu0.domain0.newidle_idx = 0
kernel.sched_domain.cpu0.domain0.wake_idx = 0
kernel.sched_domain.cpu0.domain1.busy_factor = 32
kernel.sched_domain.cpu0.domain1.busy_idx = 2
kernel.sched_domain.cpu0.domain1.cache_nice_tries = 1
kernel.sched_domain.cpu0.domain1.flags = 559
kernel.sched_domain.cpu0.domain1.forkexec_idx = 0
kernel.sched_domain.cpu0.domain1.idle_idx = 0
kernel.sched_domain.cpu0.domain1.imbalance_pct = 117
kernel.sched_domain.cpu0.domain1.max_interval = 8
kernel.sched_domain.cpu0.domain1.max_newidle_lb_cost = 39188
kernel.sched_domain.cpu0.domain1.min_interval = 4
kernel.sched_domain.cpu0.domain1.name = MC
kernel.sched_domain.cpu0.domain1.newidle_idx = 0
kernel.sched_domain.cpu0.domain1.wake_idx = 0
kernel.sched_latency_ns = 18000000
kernel.sched_migration_cost_ns = 500000
kernel.sched_min_granularity_ns = 2250000
kernel.sched_nr_migrate = 32
kernel.sched_rr_timeslice_ms = 25
kernel.sched_rt_period_us = 1000000
kernel.sched_rt_runtime_us = 950000
kernel.sched_shares_window_ns = 10000000
kernel.sched_time_avg_ms = 1000
kernel.sched_tunable_scaling = 1
kernel.sched_wakeup_granularity_ns = 3000000
kernel.secure_boot = 0
kernel.sem = 250	32000	32	128
kernel.sem_next_id = -1
kernel.sg-big-buff = 32768
kernel.shm_next_id = -1
kernel.shm_rmid_forced = 0
kernel.shmall = 18446744073692774399
kernel.shmmax = 18446744073692774399
kernel.shmmni = 4096
kernel.softlockup_all_cpu_backtrace = 0
kernel.softlockup_panic = 0
kernel.stack_tracer_enabled = 0
kernel.sysctl_writes_strict = 0
kernel.sysrq = 176
kernel.tainted = 512
kernel.threads-max = 63258
kernel.timer_migration = 1
kernel.traceoff_on_warning = 0
kernel.unknown_nmi_panic = 0
kernel.unprivileged_userns_clone = 1
kernel.version = #99~14.04.1-Ubuntu SMP Tue Jun 28 19:17:10 UTC 2016
kernel.watchdog = 1
kernel.watchdog_thresh = 10
kernel.yama.ptrace_scope = 1
```
## net.core
### rmem_default
The default setting of the socket receive buffer in bytes. This sets the default OS receive buffer size for all types of connections.

### rmem_max
The maximum receive socket buffer size in bytes. This sets the max OS receive buffer size for all types of connections.

### wmem_default
The default setting (in bytes) of the socket send buffer. This sets the max OS send buffer size for all types of connections.

### wmem_max
The maximum send socket buffer size in bytes. This sets the max OS receive buffer size for all types of connections.

### somaxconn
Limit of socket listen() backlog, known in userspace as SOMAXCONN. Defaults to 128.  See also tcp_max_syn_backlog for additional tuning for TCP sockets.

#### 注：
在linux2.2后，选择第二种方式实现，SYN_RECEIVED队列的大小由proc/sys/net/ipv4/tcp_max_syn_backlog系统参数指定，ESTABLISHED队列由backlog和/proc/sys/net/core/somaxconn中较小的指定。


```
net.core.bpf_jit_enable = 0
net.core.busy_poll = 0
net.core.busy_read = 0
net.core.default_qdisc = pfifo_fast
net.core.dev_weight = 64
net.core.flow_limit_cpu_bitmap = 00000000,00000000,00000000,00000000,00000000,00000000,00000000,00000000
net.core.flow_limit_table_len = 4096
net.core.max_skb_frags = 17
net.core.message_burst = 10
net.core.message_cost = 5
net.core.netdev_budget = 300
net.core.netdev_max_backlog = 1000
net.core.netdev_tstamp_prequeue = 1
net.core.optmem_max = 20480
net.core.rmem_default = 212992
net.core.rmem_max = 212992
net.core.rps_sock_flow_entries = 0
net.core.somaxconn = 128
net.core.warnings = 1
net.core.wmem_default = 212992
net.core.wmem_max = 212992
net.core.xfrm_acq_expires = 30
net.core.xfrm_aevent_etime = 10
net.core.xfrm_aevent_rseqth = 2
net.core.xfrm_larval_drop = 1
```
## net.ipv4

### arp_accept
BOOLEAN. Define behavior for gratuitous ARP frames who's IP is not already present in the ARP table: 
0. don't create new entries in the ARP table
1. create new entries in the ARP table

### icmp_echo_ignore_all
BOOLEAN. If set non-zero, then the kernel will ignore all ICMP ECHO 	requests sent to it.

Default: 0

### icmp_echo_ignore_broadcasts
BOOLEAN if set non-zero, then the kernel will ignore all ICMP ECHO and TIMESTAMP requests sent to it via broadcast/multicast.

Default: 1

### ip_default_ttl
Default value of TTL field (Time To Live) for outgoing (but not forwarded) IP packets. Should be between 1 and 255 inclusive. Default: 64 (as recommended by RFC1700)

### ip_local_port_range
2 INTEGERS. Defines the local port range that is used by TCP and UDP to choose the local port. The first number is the first, the second the last local port number. If possible, it is better these numbers have different parity. (one even and one odd values) The default values are 32768 and 60999 respectively.

### ip_local_reserved_ports
 list of comma separated ranges Specify the ports which are reserved for known third-party applications. These ports will not be used by automatic port
assignments (e.g. when calling connect() or bind() with port number 0). Explicit port allocation behavior is unchanged. The format used for both input and output is a comma separated list of ranges (e.g. "1,2-4,10-10" for ports 1, 2, 3, 4 and 10). Writing to the file will clear all previously reserved ports and update the current list with the one given in the input.

Note that ip_local_port_range and ip_local_reserved_ports settings are independent and both are considered by the kernel when determining which ports are available for automatic port assignments.

You can reserve ports which are not in the current ip_local_port_range, e.g.:

$ cat /proc/sys/net/ipv4/ip_local_port_range
32000	60999
$ cat /proc/sys/net/ipv4/ip_local_reserved_ports
8080,9148

although this is redundant. However such a setting is useful if later the port range is changed to a value that will include the reserved ports.

Default: Empty


### neigh/default/gc_thresh1
INTEGER. Minimum number of entries to keep. Garbage collector will not purge entries if there are fewer than this number. Default: 128

The minimum number of entries to keep in the ARP cache. The garbage collector will not run if there are fewer than this number of entries in the cache. Defaults to 128.

### neigh/default/gc_thresh2
INTEGER. Threshold when garbage collector becomes more aggressive about purging entries. Entries older than 5 seconds will be cleared when over this number. Default: 512

The soft maximum number of entries to keep in the ARP cache. The garbage collector will allow the number of entries to exceed this for 5 seconds before collection will be performed. Defaults to 512.

### neigh/default/gc_thresh3
INTEGER Maximum number of neighbor entries allowed.  Increase this when using large numbers of interfaces and when communicating with large numbers of directly-connected peers. Default: 1024

The hard maximum number of entries to keep in the ARP cache. The garbage collector will always run if there are more than this number of entries in the cache. Defaults to 1024.

### gc_interval
How frequently the garbage collector for neighbor entries should attempt to run. Defaults to 30 seconds.

### gc_stale_time
Determines how often to check for stale neighbor entries. When a neighbor entry is considered stale, it is resolved again before sending data to it. Defaults to 60 seconds.

### net.ipv4.route.max_size
INTEGER. Maximum number of routes allowed in the kernel.  Increase this when using large numbers of interfaces and/or routes. From linux kernel 3.6 onwards, this is deprecated for ipv4 as route cache is no longer used.

### min_adv_mss
NTEGER. The advertised MSS depends on the first hop route MTU, but will never be lower than this setting.

### min_pmtu
INTEGER. default 552 - minimum discovered Path MTU

### tcp_abort_on_overflow
BOOLEAN. If listening service is too slow to accept new connections, reset them. Default state is FALSE. It means that if overflow occurred due to a burst, connection will recover. Enable this option _only_ if you are really sure that listening daemon	cannot be tuned to accept connections faster. Enabling this option can harm clients of your server.

### tcp_adv_win_scale
INTEGER. Count buffering overhead as bytes/2^tcp_adv_win_scale (if tcp_adv_win_scale > 0) or bytes-bytes/2^(-tcp_adv_win_scale), if it is <= 0.	Possible values are [-31, 31], inclusive. Default: 1

### tcp_dsack
BOOLEAN Allows TCP to send "duplicate" SACKs.

### tcp_ecn  
Control use of Explicit Congestion Notification (ECN) by TCP. ECN is used only when both ends of the TCP connection indicatesupport for it.  This feature is useful in avoiding losses due to congestion by allowing supporting routers to signal congestion before having to drop packets.
Possible values are:

0 Disable ECN.  Neither initiate nor accept ECN.

1 Enable ECN when requested by incoming connections and also request ECN on outgoing connection attempts.

2 Enable ECN when requested by incoming connections but do not request ECN on outgoing connections.

Default: 2

### tcp_fastopen 
INTEGER. Enable TCP Fast Open (RFC7413) to send and accept data in the opening SYN packet.

The client support is enabled by flag 0x1 (on by default). The client then must use sendmsg() or sendto() with the MSG_FASTOPEN flag, rather than connect() to send data in SYN.

The server support is enabled by flag 0x2 (off by default). Then either enable for all listeners with another flag (0x400) or enable individual listeners via TCP_FASTOPEN socket option with the option value being the length of the syn-data backlog.

The values (bitmap) are

  0x1: (client) enables sending data in the opening SYN on the client.

  0x2: (server) enables the server support, i.e., allowing data in
		a SYN packet to be accepted and passed to the
		application before 3-way handshake finishes.

  0x4: (client) send data in the opening SYN regardless of cookie
		availability and without a cookie option.
		
  0x200: (server) accept data-in-SYN w/o any cookie option present.

  0x400: (server) enable all listeners to support Fast Open by default without explicit TCP_FASTOPEN socket option.

Default: 0x1

Note that that additional client or server features are only effective if the basic support (0x1 and 0x2) are enabled respectively.

### tcp_fin_timeout
INTEGER. The length of time an orphaned (no longer referenced by any application) connection will remain in the FIN_WAIT_2 state before it is aborted at the local end.  While a perfectly valid "receive only" state for an un-orphaned connection, an orphaned connection in FIN_WAIT_2 state could otherwise wait forever for the remote to close its end of the connection.	Cf. tcp_max_orphans

Default: 60 seconds

This specifies how many seconds to wait for a final FIN packet before the socket is forcibly closed. This is strictly a violation of the TCP specification, but required to prevent denial-of-service attacks. In Linux 2.2, the default value was 180.


### tcp_keepalive_intvl
INTEGER. The maximum number of TCP keep-alive probes to send before giving up and killing the connection if no response is obtained from the other end.

### tcp_keepalive_probes
INTEGER. How many keepalive probes TCP sends out, until it decides that the connection is broken. Default value: 9.

### tcp_keepalive_time
INTEGER. How often TCP sends out keepalive messages when keepalive is enabled. Default: 2hours.

### tcp_limit_output_bytes
INTEGER. Controls TCP Small Queue limit per tcp socket. TCP bulk sender tends to increase packets in flight until it gets losses notifications. With SNDBUF autotuning, this can result in a large amount of packets queued on the local machine (e.g.: qdiscs, CPU backlog, or device) hurting latency of other flows, for typical pfifo_fast qdiscs.  tcp_limit_output_bytes limits the number of bytes on qdisc or device to reduce artificial RTT/cwnd and reduce bufferbloat. Default: 262144

### tcp_max_orphans
INTEGER. Maximal number of TCP sockets not attached to any user file handle, held by system. If this number is exceeded orphaned connections are reset immediately and warning is printed. This limit exists only to prevent simple DoS attacks, you _must_ not rely on this or lower the limit artificially, but rather increase it (probably, after increasing installed memory), if network conditions require more than default value, and tune network services to linger and kill such states more aggressively. Let me to remind again: each orphan eats up to ~64K of unswappable memory.

### tcp_max_syn_backlog
INTEGER. Maximal number of remembered connection requests, which have not received an acknowledgment from connecting client. The minimal value is 128 for low memory machines, and it will increase in proportion to the memory of machine.	If server suffers from overload, try increasing this number.

### tcp_max_tw_buckets
INTEGER. Maximal number of timewait sockets held by system simultaneously. If this number is exceeded time-wait socket is immediately destroyed and warning is printed. This limit exists only to prevent simple DoS attacks, you _must_ not lower the limit artificially, but rather increase it (probably, after increasing installed memory), if network conditions require more than default value.

### tcp_retries1
INTEGER. This value influences the time, after which TCP decides, that something is wrong due to unacknowledged RTO retransmissions, and reports this suspicion to the network layer. See tcp_retries2 for more details. RFC 1122 recommends at least 3 retransmissions, which is the default.

一旦重传超过阈值tcp_retries1，主要的动作就是更新路由缓存。 用以避免由于路由选路变化带来的问题。

### tcp_retries2
INTEGER. This value influences the timeout of an alive TCP connection, when RTO retransmissions remain unacknowledged. Given a value of N, a hypothetical TCP connection following exponential backoff with an initial RTO of TCP_RTO_MIN would retransmit N times before killing the connection at the (N+1)th RTO. The default value of 15 yields a hypothetical timeout of 924.6	seconds and is a lower bound for the effective timeout.	TCP will effectively time out at the first RTO which exceeds the hypothetical timeout. RFC 1122 recommends at least 100 seconds for the timeout, which corresponds to a value of at least 8.

会直接放弃重传，关闭TCP流

### tcp_retries1 & tcp_retries2
Linux并不是直接拿tcp_retries1和tcp_retries2来限制重传次数的，而是用计算得到的一个timeout值来判断是否要放弃重传的。真正的重传次数同时与RTT相关。

### tcp_orphan_retries
INTEGER. This value influences the timeout of a locally closed TCP connection, when RTO retransmissions remain unacknowledged. See tcp_retries2 for more details. The default value is 8. If your machine is a loaded WEB server, you should think about lowering this value, such sockets may consume significant resources. Cf. tcp_max_orphans.

### tcp_syn_retries
INTEGER. Number of times initial SYNs for an active TCP connection attempt will be retransmitted. Should not be higher than 127. Default value is 6, which corresponds to 63 seconds till the last retransmission with the current initial RTO of 1 second. With this the final timeout for an active TCP connection attempt will happen after 127 seconds.

### tcp_synack_retries
INTEGER. Number of times SYNACKs for a passive TCP connection attempt will be retransmitted. Should not be higher than 255. Default value is 5, which corresponds to 31 seconds till the last retransmission with the current initial RTO of 1 second. With this the final timeout for a passive TCP connection will happen after 63 seconds.


### tcp_syncookies
BOOLEAN. Only valid when the kernel was compiled with CONFIG_SYN_COOKIES Send out syncookies when the syn backlog queue of a socket overflows. This is to prevent against the common 'SYN flood attack' Default: 1

Note, that syncookies is fallback facility. It MUST NOT be used to help highly loaded servers to stand against legal connection rate. If you see SYN flood warnings in your logs, but investigation	shows that they occur because of overload with legal connections, you should tune another parameters until this warning disappear. See: tcp_max_syn_backlog, tcp_synack_retries, tcp_abort_on_overflow.

syncookies seriously violate TCP protocol, **do not allow to use TCP extensions**, can result in serious degradation of some services (f.e. SMTP relaying), visible not by you, but your clients and relays, contacting you. While you see SYN flood warnings in logs not being really flooded, your server is seriously misconfigured.

If you want to test which effects syncookies have to your network connections you can set this knob to 2 to enable unconditionally generation of syncookies.

### tcp_timestamps
INTEGER. Enable timestamps as defined in RFC1323.
- 0: Disabled.
- 1: Enable timestamps as defined in RFC1323 and use random offset for each connection rather than only using the current time.
- 2: Like 1, but without random offsets.
Default: 1

### tcp_tw_recycle
(Boolean; default: disabled; since Linux 2.4) Enable fast recycling of TIME_WAIT sockets. Enabling this option is not recommended since this causes problems when working with NAT (Network Address Translation).

因为PAWS机制，所以不可以在NAT环境下设置为true。PAWS机制要求所有来个同一个host IP的TCP数据包的timestamp值是递增的。当收到一个timestamp值，小于服务端记录的对应值后，则会认为这是一个过期的数据包，然后会将其丢弃。

PAWS参考文献：
1. http://perthcharles.github.io/2015/08/27/timestamp-NAT
2. https://www.freesoft.org/CIE/RFC/1323/13.htm
3. https://www.ietf.org/rfc/rfc1323.txt


### tcp_tw_reuse - INTEGER
Enable reuse of TIME-WAIT sockets for new connections when it is safe from protocol viewpoint.
- 0 - disable
- 1 - global enable
- 2 - enable for loopback traffic only

It should not be changed without advice/request of technical
experts. Default: 2

### tcp_window_scaling
BOOLEAN. Enable window scaling as defined in RFC1323.

### xfrm4_gc_thresh
INTEGER. The threshold at which we will start garbage collecting for IPv4 destination cache entries. At twice this value the system will refuse new allocations.

### tcp_fack
tcp_fack (Boolean; default: enabled; since Linux 2.2)
Enable TCP Forward Acknowledgement support.

### tcp_sack
tcp_sack (Boolean; default: enabled; since Linux 2.2)
Enable RFC 2018 TCP Selective Acknowledgements.

### tcp_syncookies 
Enable TCP syncookies. The kernel must be compiled with CONFIG_SYN_COOKIES. Send out syncookies when the syn backlog queue of a socket overflows. The syncookies feature attempts to protect a socket from a SYN flood attack. This should be used as a last resort, if at all. This is a violation of the TCP protocol, and conflicts with other areas of TCP such as TCP extensions. It can cause problems for clients and relays. It is not recommended as a tuning mechanism for heavily loaded servers to help with overloaded or misconfigured conditions. For recommended alternatives see tcp_max_syn_backlog, tcp_synack_retries, and tcp_abort_on_overflow. 

### tcp_abort_on_overflow
Enable resetting connections if the listening service is too slow and unable to keep up and accept them. It means that if overflow occurred due to a burst, the connection will recover. Enable this option only if you are really sure that the listening daemon cannot be tuned to accept connections faster. Enabling this option can harm the clients of your server.

```
net.ipv4.cipso_cache_bucket_size = 10
net.ipv4.cipso_cache_enable = 1
net.ipv4.cipso_rbm_optfmt = 0
net.ipv4.cipso_rbm_strictvalid = 1
net.ipv4.conf.all.accept_local = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.arp_accept = 0
net.ipv4.conf.all.arp_announce = 0
net.ipv4.conf.all.arp_filter = 0
net.ipv4.conf.all.arp_ignore = 0
net.ipv4.conf.all.arp_notify = 0
net.ipv4.conf.all.bootp_relay = 0
net.ipv4.conf.all.disable_policy = 0
net.ipv4.conf.all.disable_xfrm = 0
net.ipv4.conf.all.force_igmp_version = 0
net.ipv4.conf.all.forwarding = 1
net.ipv4.conf.all.igmpv2_unsolicited_report_interval = 10000
net.ipv4.conf.all.igmpv3_unsolicited_report_interval = 1000
net.ipv4.conf.all.log_martians = 0
net.ipv4.conf.all.mc_forwarding = 0
net.ipv4.conf.all.medium_id = 0
net.ipv4.conf.all.promote_secondaries = 0
net.ipv4.conf.all.proxy_arp = 0
net.ipv4.conf.all.proxy_arp_pvlan = 0
net.ipv4.conf.all.route_localnet = 0
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.all.secure_redirects = 1
net.ipv4.conf.all.send_redirects = 1
net.ipv4.conf.all.shared_media = 1
net.ipv4.conf.all.src_valid_mark = 0
net.ipv4.conf.all.tag = 0
net.ipv4.conf.default.accept_local = 0
net.ipv4.conf.default.accept_redirects = 1
net.ipv4.conf.default.accept_source_route = 1
net.ipv4.conf.default.arp_accept = 0
net.ipv4.conf.default.arp_announce = 0
net.ipv4.conf.default.arp_filter = 0
net.ipv4.conf.default.arp_ignore = 0
net.ipv4.conf.default.arp_notify = 0
net.ipv4.conf.default.bootp_relay = 0
net.ipv4.conf.default.disable_policy = 0
net.ipv4.conf.default.disable_xfrm = 0
net.ipv4.conf.default.force_igmp_version = 0
net.ipv4.conf.default.forwarding = 1
net.ipv4.conf.default.igmpv2_unsolicited_report_interval = 10000
net.ipv4.conf.default.igmpv3_unsolicited_report_interval = 1000
net.ipv4.conf.default.log_martians = 0
net.ipv4.conf.default.mc_forwarding = 0
net.ipv4.conf.default.medium_id = 0
net.ipv4.conf.default.promote_secondaries = 0
net.ipv4.conf.default.proxy_arp = 0
net.ipv4.conf.default.proxy_arp_pvlan = 0
net.ipv4.conf.default.route_localnet = 0
net.ipv4.conf.default.rp_filter = 1
net.ipv4.conf.default.secure_redirects = 1
net.ipv4.conf.default.send_redirects = 1
net.ipv4.conf.default.shared_media = 1
net.ipv4.conf.default.src_valid_mark = 0
net.ipv4.conf.default.tag = 0
net.ipv4.conf.eth0.accept_local = 0
net.ipv4.conf.eth0.accept_redirects = 1
net.ipv4.conf.eth0.accept_source_route = 1
net.ipv4.conf.eth0.arp_accept = 0
net.ipv4.conf.eth0.arp_announce = 0
net.ipv4.conf.eth0.arp_filter = 0
net.ipv4.conf.eth0.arp_ignore = 0
net.ipv4.conf.eth0.arp_notify = 0
net.ipv4.conf.eth0.bootp_relay = 0
net.ipv4.conf.eth0.disable_policy = 0
net.ipv4.conf.eth0.disable_xfrm = 0
net.ipv4.conf.eth0.force_igmp_version = 0
net.ipv4.conf.eth0.forwarding = 1
net.ipv4.conf.eth0.igmpv2_unsolicited_report_interval = 10000
net.ipv4.conf.eth0.igmpv3_unsolicited_report_interval = 1000
net.ipv4.conf.eth0.log_martians = 0
net.ipv4.conf.eth0.mc_forwarding = 0
net.ipv4.conf.eth0.medium_id = 0
net.ipv4.conf.eth0.promote_secondaries = 0
net.ipv4.conf.eth0.proxy_arp = 0
net.ipv4.conf.eth0.proxy_arp_pvlan = 0
net.ipv4.conf.eth0.route_localnet = 0
net.ipv4.conf.eth0.rp_filter = 1
net.ipv4.conf.eth0.secure_redirects = 1
net.ipv4.conf.eth0.send_redirects = 1
net.ipv4.conf.eth0.shared_media = 1
net.ipv4.conf.eth0.src_valid_mark = 0
net.ipv4.conf.eth0.tag = 0
net.ipv4.fwmark_reflect = 0
net.ipv4.icmp_echo_ignore_all = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.icmp_errors_use_inbound_ifaddr = 0
net.ipv4.icmp_ignore_bogus_error_responses = 1
net.ipv4.icmp_ratelimit = 1000
net.ipv4.icmp_ratemask = 6168
net.ipv4.igmp_max_memberships = 20
net.ipv4.igmp_max_msf = 10
net.ipv4.inet_peer_maxttl = 600
net.ipv4.inet_peer_minttl = 120
net.ipv4.inet_peer_threshold = 65664
net.ipv4.ip_default_ttl = 64
net.ipv4.ip_dynaddr = 1
net.ipv4.ip_early_demux = 1
net.ipv4.ip_forward = 1
net.ipv4.ip_forward_use_pmtu = 0
net.ipv4.ip_local_port_range = 32768	61000
net.ipv4.ip_local_reserved_ports = 
net.ipv4.ip_no_pmtu_disc = 0
net.ipv4.ip_nonlocal_bind = 0
net.ipv4.ipfrag_high_thresh = 4194304
net.ipv4.ipfrag_low_thresh = 3145728
net.ipv4.ipfrag_max_dist = 64
net.ipv4.ipfrag_secret_interval = 600
net.ipv4.ipfrag_time = 30
net.ipv4.neigh.default.anycast_delay = 100
net.ipv4.neigh.default.app_solicit = 0
net.ipv4.neigh.default.base_reachable_time_ms = 30000
net.ipv4.neigh.default.delay_first_probe_time = 5
net.ipv4.neigh.default.gc_interval = 30
net.ipv4.neigh.default.gc_stale_time = 60
net.ipv4.neigh.default.gc_thresh1 = 128
net.ipv4.neigh.default.gc_thresh2 = 512
net.ipv4.neigh.default.gc_thresh3 = 1024
net.ipv4.neigh.default.locktime = 100
net.ipv4.neigh.default.mcast_solicit = 3
net.ipv4.neigh.default.proxy_delay = 80
net.ipv4.neigh.default.proxy_qlen = 64
net.ipv4.neigh.default.retrans_time_ms = 1000
net.ipv4.neigh.default.ucast_solicit = 3
net.ipv4.neigh.default.unres_qlen = 31
net.ipv4.neigh.default.unres_qlen_bytes = 65536
net.ipv4.neigh.eth0.anycast_delay = 100
net.ipv4.neigh.eth0.app_solicit = 0
net.ipv4.neigh.eth0.base_reachable_time_ms = 30000
net.ipv4.neigh.eth0.delay_first_probe_time = 5
net.ipv4.neigh.eth0.gc_stale_time = 60
net.ipv4.neigh.eth0.locktime = 100
net.ipv4.neigh.eth0.mcast_solicit = 3
net.ipv4.neigh.eth0.proxy_delay = 80
net.ipv4.neigh.eth0.proxy_qlen = 64
net.ipv4.neigh.eth0.retrans_time_ms = 1000
net.ipv4.neigh.eth0.ucast_solicit = 3
net.ipv4.neigh.eth0.unres_qlen = 31
net.ipv4.neigh.eth0.unres_qlen_bytes = 65536
net.ipv4.ping_group_range = 1	0
net.ipv4.route.error_burst = 1250
net.ipv4.route.error_cost = 250
net.ipv4.route.gc_elasticity = 8
net.ipv4.route.gc_interval = 60
net.ipv4.route.gc_min_interval = 0
net.ipv4.route.gc_min_interval_ms = 500
net.ipv4.route.gc_thresh = -1
net.ipv4.route.gc_timeout = 300
net.ipv4.route.max_size = 2147483647
net.ipv4.route.min_adv_mss = 256
net.ipv4.route.min_pmtu = 552
net.ipv4.route.mtu_expires = 600
net.ipv4.route.redirect_load = 5
net.ipv4.route.redirect_number = 9
net.ipv4.route.redirect_silence = 5120
net.ipv4.tcp_abort_on_overflow = 0
net.ipv4.tcp_adv_win_scale = 1
net.ipv4.tcp_allowed_congestion_control = cubic reno
net.ipv4.tcp_app_win = 31
net.ipv4.tcp_autocorking = 1
net.ipv4.tcp_available_congestion_control = cubic reno
net.ipv4.tcp_base_mss = 512
net.ipv4.tcp_challenge_ack_limit = 100
net.ipv4.tcp_congestion_control = cubic
net.ipv4.tcp_dsack = 1
net.ipv4.tcp_early_retrans = 3
net.ipv4.tcp_ecn = 2
net.ipv4.tcp_fack = 1
net.ipv4.tcp_fastopen = 1
net.ipv4.tcp_fin_timeout = 60
net.ipv4.tcp_frto = 2
net.ipv4.tcp_fwmark_accept = 0
net.ipv4.tcp_keepalive_intvl = 75
net.ipv4.tcp_keepalive_probes = 9
net.ipv4.tcp_keepalive_time = 7200
net.ipv4.tcp_limit_output_bytes = 131072
net.ipv4.tcp_low_latency = 0
net.ipv4.tcp_max_orphans = 32768
net.ipv4.tcp_max_syn_backlog = 256
net.ipv4.tcp_max_tw_buckets = 32768
net.ipv4.tcp_mem = 187398	249866	374796
net.ipv4.tcp_min_tso_segs = 2
net.ipv4.tcp_moderate_rcvbuf = 1
net.ipv4.tcp_mtu_probing = 0
net.ipv4.tcp_no_metrics_save = 0
net.ipv4.tcp_notsent_lowat = -1
net.ipv4.tcp_orphan_retries = 0
net.ipv4.tcp_reordering = 3
net.ipv4.tcp_retrans_collapse = 1
net.ipv4.tcp_retries1 = 3
net.ipv4.tcp_retries2 = 15
net.ipv4.tcp_rfc1337 = 0
net.ipv4.tcp_rmem = 4096	87380	6291456
net.ipv4.tcp_sack = 1
net.ipv4.tcp_slow_start_after_idle = 1
net.ipv4.tcp_stdurg = 0
net.ipv4.tcp_syn_retries = 6
net.ipv4.tcp_synack_retries = 5
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_thin_dupack = 0
net.ipv4.tcp_thin_linear_timeouts = 0
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_tso_win_divisor = 3
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_tw_reuse = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096	16384	4194304
net.ipv4.tcp_workaround_signed_windows = 0
net.ipv4.udp_mem = 187398	249866	374796
net.ipv4.udp_rmem_min = 4096
net.ipv4.udp_wmem_min = 4096
net.ipv4.xfrm4_gc_thresh = 32768
```
## net.ipv6
```
net.ipv6.anycast_src_echo_reply = 0
net.ipv6.bindv6only = 0
net.ipv6.conf.all.accept_dad = 1
net.ipv6.conf.all.accept_ra = 1
net.ipv6.conf.all.accept_ra_defrtr = 1
net.ipv6.conf.all.accept_ra_min_hop_limit = 1
net.ipv6.conf.all.accept_ra_pinfo = 1
net.ipv6.conf.all.accept_ra_rt_info_max_plen = 0
net.ipv6.conf.all.accept_ra_rtr_pref = 1
net.ipv6.conf.all.accept_redirects = 1
net.ipv6.conf.all.accept_source_route = 0
net.ipv6.conf.all.autoconf = 1
net.ipv6.conf.all.dad_transmits = 1
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.all.force_mld_version = 0
net.ipv6.conf.all.force_tllao = 0
net.ipv6.conf.all.forwarding = 0
net.ipv6.conf.all.hop_limit = 64
net.ipv6.conf.all.max_addresses = 16
net.ipv6.conf.all.max_desync_factor = 600
net.ipv6.conf.all.mc_forwarding = 0
net.ipv6.conf.all.mldv1_unsolicited_report_interval = 10000
net.ipv6.conf.all.mldv2_unsolicited_report_interval = 1000
net.ipv6.conf.all.mtu = 1280
net.ipv6.conf.all.ndisc_notify = 0
net.ipv6.conf.all.proxy_ndp = 0
net.ipv6.conf.all.regen_max_retry = 3
net.ipv6.conf.all.router_probe_interval = 60
net.ipv6.conf.all.router_solicitation_delay = 1
net.ipv6.conf.all.router_solicitation_interval = 4
net.ipv6.conf.all.router_solicitations = 3
net.ipv6.conf.all.suppress_frag_ndisc = 1
net.ipv6.conf.all.temp_prefered_lft = 86400
net.ipv6.conf.all.temp_valid_lft = 604800
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.accept_dad = 1
net.ipv6.conf.default.accept_ra = 1
net.ipv6.conf.default.accept_ra_defrtr = 1
net.ipv6.conf.default.accept_ra_min_hop_limit = 1
net.ipv6.conf.default.accept_ra_pinfo = 1
net.ipv6.conf.default.accept_ra_rt_info_max_plen = 0
net.ipv6.conf.default.accept_ra_rtr_pref = 1
net.ipv6.conf.default.accept_redirects = 1
net.ipv6.conf.default.accept_source_route = 0
net.ipv6.conf.default.autoconf = 1
net.ipv6.conf.default.dad_transmits = 1
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.default.force_mld_version = 0
net.ipv6.conf.default.force_tllao = 0
net.ipv6.conf.default.forwarding = 0
net.ipv6.conf.default.hop_limit = 64
net.ipv6.conf.default.max_addresses = 16
net.ipv6.conf.default.max_desync_factor = 600
net.ipv6.conf.default.mc_forwarding = 0
net.ipv6.conf.default.mldv1_unsolicited_report_interval = 10000
net.ipv6.conf.default.mldv2_unsolicited_report_interval = 1000
net.ipv6.conf.default.mtu = 1280
net.ipv6.conf.default.ndisc_notify = 0
net.ipv6.conf.default.proxy_ndp = 0
net.ipv6.conf.default.regen_max_retry = 3
net.ipv6.conf.default.router_probe_interval = 60
net.ipv6.conf.default.router_solicitation_delay = 1
net.ipv6.conf.default.router_solicitation_interval = 4
net.ipv6.conf.default.router_solicitations = 3
net.ipv6.conf.default.suppress_frag_ndisc = 1
net.ipv6.conf.default.temp_prefered_lft = 86400
net.ipv6.conf.default.temp_valid_lft = 604800
net.ipv6.conf.default.use_tempaddr = 2
net.ipv6.conf.eth0.accept_dad = 1
net.ipv6.conf.eth0.accept_ra = 0
net.ipv6.conf.eth0.accept_ra_defrtr = 1
net.ipv6.conf.eth0.accept_ra_min_hop_limit = 1
net.ipv6.conf.eth0.accept_ra_pinfo = 1
net.ipv6.conf.eth0.accept_ra_rt_info_max_plen = 0
net.ipv6.conf.eth0.accept_ra_rtr_pref = 1
net.ipv6.conf.eth0.accept_redirects = 1
net.ipv6.conf.eth0.accept_source_route = 0
net.ipv6.conf.eth0.autoconf = 1
net.ipv6.conf.eth0.dad_transmits = 1
net.ipv6.conf.eth0.disable_ipv6 = 0
net.ipv6.conf.eth0.force_mld_version = 0
net.ipv6.conf.eth0.force_tllao = 0
net.ipv6.conf.eth0.forwarding = 0
net.ipv6.conf.eth0.hop_limit = 64
net.ipv6.conf.eth0.max_addresses = 16
net.ipv6.conf.eth0.max_desync_factor = 600
net.ipv6.conf.eth0.mc_forwarding = 0
net.ipv6.conf.eth0.mldv1_unsolicited_report_interval = 10000
net.ipv6.conf.eth0.mldv2_unsolicited_report_interval = 1000
net.ipv6.conf.eth0.mtu = 1500
net.ipv6.conf.eth0.ndisc_notify = 0
net.ipv6.conf.eth0.proxy_ndp = 0
net.ipv6.conf.eth0.regen_max_retry = 3
net.ipv6.conf.eth0.router_probe_interval = 60
net.ipv6.conf.eth0.router_solicitation_delay = 1
net.ipv6.conf.eth0.router_solicitation_interval = 4
net.ipv6.conf.eth0.router_solicitations = 3
net.ipv6.conf.eth0.suppress_frag_ndisc = 1
net.ipv6.conf.eth0.temp_prefered_lft = 86400
net.ipv6.conf.eth0.temp_valid_lft = 604800
net.ipv6.conf.eth0.use_tempaddr = 0
net.ipv6.flowlabel_consistency = 1
net.ipv6.fwmark_reflect = 0
net.ipv6.icmp.ratelimit = 1000
net.ipv6.ip6frag_high_thresh = 4194304
net.ipv6.ip6frag_low_thresh = 3145728
net.ipv6.ip6frag_secret_interval = 600
net.ipv6.ip6frag_time = 60
net.ipv6.mld_max_msf = 64
net.ipv6.neigh.default.anycast_delay = 100
net.ipv6.neigh.default.app_solicit = 0
net.ipv6.neigh.default.base_reachable_time_ms = 30000
net.ipv6.neigh.default.delay_first_probe_time = 5
net.ipv6.neigh.default.gc_interval = 30
net.ipv6.neigh.default.gc_stale_time = 60
net.ipv6.neigh.default.gc_thresh1 = 128
net.ipv6.neigh.default.gc_thresh2 = 512
net.ipv6.neigh.default.gc_thresh3 = 1024
net.ipv6.neigh.default.locktime = 0
net.ipv6.neigh.default.mcast_solicit = 3
net.ipv6.neigh.default.proxy_delay = 80
net.ipv6.neigh.default.proxy_qlen = 64
net.ipv6.neigh.default.retrans_time_ms = 1000
net.ipv6.neigh.default.ucast_solicit = 3
net.ipv6.neigh.default.unres_qlen = 31
net.ipv6.neigh.default.unres_qlen_bytes = 65536
net.ipv6.neigh.eth0.anycast_delay = 100
net.ipv6.neigh.eth0.app_solicit = 0
net.ipv6.neigh.eth0.base_reachable_time_ms = 30000
net.ipv6.neigh.eth0.delay_first_probe_time = 5
net.ipv6.neigh.eth0.gc_stale_time = 60
net.ipv6.neigh.eth0.locktime = 0
net.ipv6.neigh.eth0.mcast_solicit = 3
net.ipv6.neigh.eth0.proxy_delay = 80
net.ipv6.neigh.eth0.proxy_qlen = 64
net.ipv6.neigh.eth0.retrans_time_ms = 1000
net.ipv6.neigh.eth0.ucast_solicit = 3
net.ipv6.neigh.eth0.unres_qlen = 31
net.ipv6.neigh.eth0.unres_qlen_bytes = 65536
net.ipv6.route.gc_elasticity = 9
net.ipv6.route.gc_interval = 30
net.ipv6.route.gc_min_interval = 0
net.ipv6.route.gc_min_interval_ms = 500
net.ipv6.route.gc_thresh = 1024
net.ipv6.route.gc_timeout = 60
net.ipv6.route.max_size = 4096
net.ipv6.route.min_adv_mss = 1220
net.ipv6.route.mtu_expires = 600
net.ipv6.xfrm6_gc_thresh = 32768
```
## net.netfilter
```
net.netfilter.nf_conntrack_acct = 0
net.netfilter.nf_conntrack_buckets = 16384
net.netfilter.nf_conntrack_checksum = 1
net.netfilter.nf_conntrack_count = 23
net.netfilter.nf_conntrack_events = 1
net.netfilter.nf_conntrack_events_retry_timeout = 15
net.netfilter.nf_conntrack_expect_max = 256
net.netfilter.nf_conntrack_generic_timeout = 600
net.netfilter.nf_conntrack_helper = 1
net.netfilter.nf_conntrack_icmp_timeout = 30
net.netfilter.nf_conntrack_log_invalid = 0
net.netfilter.nf_conntrack_max = 65536
net.netfilter.nf_conntrack_tcp_be_liberal = 0
net.netfilter.nf_conntrack_tcp_loose = 1
net.netfilter.nf_conntrack_tcp_max_retrans = 3
net.netfilter.nf_conntrack_tcp_timeout_close = 10
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_tcp_timeout_established = 432000
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_last_ack = 30
net.netfilter.nf_conntrack_tcp_timeout_max_retrans = 300
net.netfilter.nf_conntrack_tcp_timeout_syn_recv = 60
net.netfilter.nf_conntrack_tcp_timeout_syn_sent = 120
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_unacknowledged = 300
net.netfilter.nf_conntrack_timestamp = 0
net.netfilter.nf_conntrack_udp_timeout = 30
net.netfilter.nf_conntrack_udp_timeout_stream = 180
net.netfilter.nf_log.0 = NONE
net.netfilter.nf_log.1 = NONE
net.netfilter.nf_log.10 = NONE
net.netfilter.nf_log.11 = NONE
net.netfilter.nf_log.12 = NONE
net.netfilter.nf_log.2 = NONE
net.netfilter.nf_log.3 = NONE
net.netfilter.nf_log.4 = NONE
net.netfilter.nf_log.5 = NONE
net.netfilter.nf_log.6 = NONE
net.netfilter.nf_log.7 = NONE
net.netfilter.nf_log.8 = NONE
net.netfilter.nf_log.9 = NONE
```
```
net.nf_conntrack_max = 65536
net.unix.max_dgram_qlen = 10
```
## vm
```
vm.admin_reserve_kbytes = 8192
vm.block_dump = 0
vm.dirty_background_bytes = 0
vm.dirty_background_ratio = 40
vm.dirty_bytes = 0
vm.dirty_expire_centisecs = 3000
vm.dirty_ratio = 60
vm.dirty_writeback_centisecs = 60000
vm.drop_caches = 0
vm.extfrag_threshold = 500
vm.hugepages_treat_as_movable = 0
vm.hugetlb_shm_group = 0
vm.laptop_mode = 5
vm.legacy_va_layout = 0
vm.lowmem_reserve_ratio = 256	256	32
vm.max_map_count = 65530
vm.memory_failure_early_kill = 0
vm.memory_failure_recovery = 1
vm.min_free_kbytes = 67584
vm.min_slab_ratio = 5
vm.min_unmapped_ratio = 1
vm.mmap_min_addr = 65536
vm.nr_hugepages = 0
vm.nr_hugepages_mempolicy = 0
vm.nr_overcommit_hugepages = 0
vm.nr_pdflush_threads = 0
vm.numa_zonelist_order = default
vm.oom_dump_tasks = 1
vm.oom_kill_allocating_task = 0
vm.overcommit_kbytes = 0
vm.overcommit_memory = 0
vm.overcommit_ratio = 50
vm.page-cluster = 3
vm.panic_on_oom = 0
vm.percpu_pagelist_fraction = 0
vm.scan_unevictable_pages = 0
vm.stat_interval = 1
vm.swappiness = 60
vm.user_reserve_kbytes = 131072
vm.vfs_cache_pressure = 100
vm.zone_reclaim_mode = 0
```


# ulimit
它具有一套参数集，用于为由它生成的**Shell 进程**及其**子进程**的资源使用设置限制。

ulimit 用于限制 shell启动进程所占用的资源，支持以下各种类型的限制：所创建的内核文件的大小、进程数据块的大小、Shell 进程创建文件的大小、内存锁住的大小、常驻内存集的大小、打开文件描述符的数量、分配堆栈的最大大小、CPU时间、单个用户的最大线程数、Shell进程所能使用的最大虚拟内存。同时，它支持硬资源和软资源的限制。

Provides control over the resources available **to the shell** and **to processes started by it**, on systems that allow such control. The -H and -S options specify that the hard or soft limit is set for the given resource. A hard limit cannot be increased by a non-root user once it is set; a soft limit may be increased up to the value of the hard limit. If neither -H nor -S is specified, both the soft and hard limits are set. The value of limit can be a number in the unit specified for the resource or one of the special values hard, soft, or unlimited, which stand for the current hard limit, the current soft limit, and no limit, respectively. If limit is omitted, the current value of the soft limit of the resource is printed, unless the -H option is given. 

## Soft Limit & Hard Limit
The soft limit, on the other hand, is a “warning” limit. It tells the user and the system admin that you are close to reach the danger level, which is the hard limit. Users are allowed to go over the soft limit, unlike the hard limit (it is an absolute no-no to go over hard limits).

# limits.conf
位于/etc/security/limits.conf。修改文件后，需要重启才能生效。针对所有用户生效。

## /etc/security/limits
该文件不仅能限制指定用户的资源使用，还能限制指定组的资源使用

## 参考文献
[通过 ulimit 改善系统性能](https://www.ibm.com/developerworks/cn/linux/l-cn-ulimit/index.html)


# 网卡参数
查看网卡支持多队列情况。运行命令：ethtool -l eth0。

设置网卡使用多队列。运行命令：ethtool -l eth0 combined 2。

对于有多个网卡的用户，可以对多个网卡分别进行设置：
```
[root@localhost ~]# ethtool -l eth0
Channel parameters for eth0:
Pre-set maximums:
RX: 0
TX: 0
Other: 0
Combined: 2 # 这一行表示最多支持设置2个队列
Current hardware settings:
RX: 0
TX: 0
Other: 0
Combined: 1 #表示当前生效的是1个队列
[root@localhost ~]# ethtool -L eth0 combined 2 # 设置eth0当前使用2个队列
```

## 参考文献
[网卡多队列](https://www.alibabacloud.com/help/zh/doc-detail/52559.htm)
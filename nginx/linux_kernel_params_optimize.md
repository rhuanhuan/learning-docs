## 部署Nginx：Linux内核参数优化

默认Linux内核参数考虑的是最通用的场景，不符合Nginx用于“支持高并发访问的Web服务器”的配置。修改Linux内核参数可以使Nginx拥有更高的性能。

内核优化跟实际的业务场景相关联，比如“作为静态Web内容服务器”，“反向代理服务器”，“实时提供缩略图功能的服务器”时，配置都不一样。

#### 使Nginx 支持更多并发请求
- 修改`/etc/sysctl.conf` 来更改内核参数。执行`sysctl -p`来使改动生效。  

例如：
```
# 表示进程可以同时打开的最大句柄数，这个参数直接限制最大并发连接数
fs.file-max = 999999

# 设置为1，表示允许TIME-WAIT状态的socket重新用于新的TCP连接。对于服务器来说很有意义，因为服务器上总会有大量的TIME-WAIT状态的连接
net.ipv4.tcp_tw_reuse = 1

# 表示当Keep Alive启动时，TCP发送keepalive消息的频率。默认是2h, 将其设置小一些可以更快地清理无效连接
net.ipv4.tcp_keepalive_time = 600

# 表示当服务器主动关闭连接时，socket保持在 FIN-WAIT-2状态的最大时间
net.ipv4.tcp_fin_timeout = 30

# 表示操作系统允许TIME_WAIT套接字数量的最大值  
# 如果超过TIME_WAIT套接字将立即被清除，并打印警告信息。  
# 该参数默认为180,000， 过多的TIME_WAIT会使Web服务器变慢。
net.ipv4.tcp_max_tw_buckets = 5000

# UDP和TCP连接中，本地端口的取值范围
net.ipv4.ip_local_port_range = 1024  61000

# TCP接受缓存（用于TCP接收滑动窗口）的最小值、默认值、最大值
net.ipv4.tcp_rmem = 4096 32768 262142

# TCP发送缓存（用于TCP发送滑动窗口）的最小值、默认值、最大值
net.ipv4.tcp_wmem = 4096 32768 262142

# 当网卡接收数据包的速度大于内核处理的速度时，会有一个队列保存这个数据包。该参数表示这个队列的最大值
net.core.netdev_max_blacklog = 8096

# 内核套接字 接受/发送 缓存区的 默认/最大 大小
net.core.rmem_default = 262144
net.core.wmem_default = 262144
net.core.rmem_max = 2097152
net.core.wmem_max = 2097152

# 该参数与性能无关，用于解决TCP的SYN攻击
net.ipv4.tcp_syncookies = 1

# TCP连接建立阶段，接收SYN请求队列的最大长度，默认为1024， 将其设置大一些可以使Nginx繁忙来不解accept连接时，Linux不会丢失客户端发出的请求
net.ipv4.tcp_max_syn_backlog = 1024
```

__滑动窗口的大小__ 与 __套接字缓存区__ 会影响并发连接的数量。每个TCP连接都会为了维护TCP滑动窗口而消耗内存，这个窗口会根据服务器的处理速度收缩或扩张。  
__wmem_max__ 的设置需要平衡 __物理内存的大小__ 以及 __Nginx并发处理的最大连接数量__(由nginx.conf中的worker_processes)，





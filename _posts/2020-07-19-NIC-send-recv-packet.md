---
layout: article
tags: NIC
title: 网卡收发包流程
mathjax: true
key: python
---

## softnet_data数据结构
```
每个cpu都有收发包的入口队列和出口队列，他们都包含在一个sofnet_data结构体中。
每个cpu有一个softnet_data结构体。

struct softnet_data
{
	int     		throttle;
	int 	cng_level;
	int	avg_blog;
	struct 	sk_buff_head 	input_pkt_queue;
	struct 	list_head 	poll_list;
	struct 	net_device 	*output_queue;
	struct 	sk_buff 	*completion_queue;
	struct 	net_device 	backlog_dev;
}

throttle,cng_level,avg_blog 用于拥塞控制
input_pkt_queue 非NPAI的网卡收到的包都放到这个队列中
poll_list 是有数据要接收的设备列表
output_queue 是有数据要发送的设备列表
completion_queue 是需要释放的发送缓冲区列表
```

## 收包流程
```
https://stackoverflow.com/questions/43696997/ring-buffers-and-dma

Network Device Receives Frames and these frames are transferred to the DMA ring buffer.
Now After making this transfer an interrupt is raised to let the CPU know that the transfer has been made.
In the interrupt handler routine the CPU transfers the data from the DMA ring buffer to the CPU network input queue for later time.
Bottom Half of the handler routine is to process the packets from the CPU network input queue and pass it to the appropriate layers.

1、网卡收到数据包后，通过DMA把报文发送到ring buffer（DMA ring buffer = NIC RX/RXring buffer）
2、网卡给CPU发送一个硬件中断
3、中断处理程序，也就是驱动程序，根据包长度分配skb_buff，并且把包（帧）拷贝到skb_buff中。（如果设备使用DMA，驱动程序只需要初始化一个指针，不需要拷贝。如果不支持DMA就多一次copy？）
	a)如果是非NAPI设备，skb_buff会放到对应的CPU的softnet_data.input_pkt_queue中。所有的非NAPI网卡公用这个相同的入口队列。
	b)如果是NAPI设备，每个网卡有自己的队列。并且网卡的poll方法（该方法在net_device中定义，由下半部softirq调用）直接从网卡的环形缓冲区(内存映射？)取数据。
	c)input_pkt_queue队列的最大长度是300，如果超出这个长度，新的包就会被丢弃。
4、中断处理程序，对一些skb_buff字段初始化，以便上层网络使用，比如skb_protocol
5、中断处理程序，调度软中断NET_RX_SOFTIRQ，把自己加入cpu.softnet_data.poll_list中，并结束
6、当软中断NET_RX_SOFTIRQ被执行时，net_rx_action函数会取出cpu.softnet_data.poll_list中的第一个设备，执行该设备的poll方法（这里只讨论NAPI）
7、poll方法大概会调用netif_receive_skb方法
8、netif_receive_skb主要完成三个任务
	a)把包的副本发送给每个分流器
	b)把包的副本发送给skb->protocol所关联的协议处理函数
	c)一些其它功能，比如bridge，bonding
9、三层协议最后可以转发、丢弃、接收这个报文。

```

## 发包流程
```
1、调用dev_queue_xmit函数
2、如果没有qdisc队列规则，直接调用hard_start_xmit方法发送数据
3、如果有qidsc队列规则，调用qdisc_enqueue（把包加入网卡队列 内存映射？）->
	qidsc_run（不停的调用qdisc_restart，直到队列停止）->
	qdisc_restart（从队列取包）->
	hard_start_xmit（由网卡发送）
4、hard_start_xmit可能成功，也可能失败，这是因为当网卡的内存剩余不足一个mtu大小时，驱动会调用netif_stop_queue停止出口队列。
   后面当内存可用时，设备会产生一个中断，中断处理程序通过调用netif_wake_queue恢复出口队列。
   失败了也没关系，因为后续软中断会被触发，继续发送。
5、两种情形触发NET_TX_SOFTIRQ
   a)netif_wak_queue->_netif_schedule->raise_softirq（此时会把设备自己加入cpu.softnet_data.output_queue中）
   b)当发送完成，且驱动程序通过dev_kfree_skb_irq通知缓冲区可以释放时(此时会把缓冲区地址加入cpu.softnet_data.completion_queue中)
6、当NET_TX_SOFTIRQ执行时，完成两个工作
   a)free_skb释放已发送完成的缓冲区
   b)执行qdisc_run->qdisc_restart->hard_start_xmit发送报文，会不停的发送，直到驱动通过netif_stop_queue停止出口队列

注意：出口队列停止后应该能在一定的时间之内恢复，看门狗定时器用于监控设备是否在给定时间内恢复，如果不能恢复会调用驱动程序提供的tx_timeout函数。
      该函数会复位网卡，然后以netif_wake_queue重启接口队列。
```

## From DeepSeek , receive process
```
在Linux系统中，网卡从接收数据包到将数据传递给应用程序的过程涉及多个步骤，主要包括硬件中断、DMA传输、内核网络协议栈处理以及最终将数据拷贝到用户空间。以下是详细步骤：
### 1. **网卡接收数据包（物理层）**
   - 数据包通过网络到达网卡（NIC）。
   - 网卡检查数据包的目的MAC地址是否匹配自己的MAC地址（或者为广播/多播地址），如果是，则接收该数据包。
### 2. **DMA传输到内核内存**
   - 网卡通过直接内存访问（DMA）将接收到的数据包直接写入到内核预留的环形缓冲区（Ring Buffer）中，这个缓冲区通常称为接收环（RX Ring）。该操作不需要CPU参与。
   - 每个数据包会被封装成一个`sk_buff`结构（即socket buffer，是Linux内核中表示网络数据包的结构体），并存入接收队列。 (这里指的应该是非NAPI用到到softnet_data -> input_pkt_queue)
### 3. **网卡触发硬件中断**
   - 当网卡将数据包写入内存后，会触发一个硬件中断（IRQ）通知CPU有新的数据包到达。
   - CPU会暂停当前任务，转而执行网卡驱动注册的中断处理函数。
### 4. **中断处理函数（上半部）**
   - 中断处理函数（上半部）主要完成以下工作：
     - 禁用网卡的中断（避免中断嵌套），并通知网卡已经收到中断。
     - 调度下半部处理程序（软中断）来处理数据包。在Linux中，通常使用软中断（softirq）机制来避免在中断上下文中处理过多任务。
     - 重新使能网卡中断（在有些实现中，可能会在软中断处理完成后才重新使能中断）。
### 5. **软中断处理（下半部）**
   - 软中断（特别是`NET_RX_SOFTIRQ`）被触发，执行网卡驱动注册的轮询函数（如`napi_poll`）。
   - 在NAPI（New API）机制下，驱动会从接收环（RX Ring）中批量读取多个数据包（减少中断次数，提高效率）。
   - 对于每个数据包，进行如下处理：
     - **数据链路层处理**：检查以太网帧头，确定上层协议（如IPv4、IPv6等），并移除以太网帧头。
     - **网络层处理**：将数据包传递给网络层协议处理函数（如`ip_rcv`）。在网络层，进行IP头校验，检查目的IP地址是否为本机，如果是则根据IP头中的协议字段（如TCP、UDP）将数据包传递给传输层处理函数。同时可能进行路由判断（如果是转发则进入转发流程，这里假设是发给本机的）。
     - **传输层处理**：例如，如果是TCP数据包，会由`tcp_v4_rcv`（或IPv6对应函数）处理。TCP层处理包括检查序列号、校验和，将数据包放入对应的socket的接收缓冲区（receive buffer）中，并更新TCP状态机。
### 6. **唤醒应用程序**
   - 当数据包被放入socket的接收缓冲区后，内核会唤醒等待在该socket上的应用程序。
   - 如果应用程序使用阻塞I/O，它会被从等待队列中移出并标记为可运行状态。
### 7. **应用程序读取数据**
   - 当应用程序执行读操作（如`read()`, `recv()`等系统调用）时，数据从内核空间的socket接收缓冲区复制到用户空间的缓冲区。
   - 如果是使用零拷贝技术（如`sendfile`），则可以避免数据在用户空间和内核空间之间的拷贝。
### 补充说明：多队列网卡与RSS
   - 现代网卡支持多队列（例如RSS, Receive Side Scaling），可以将不同流的数据包分发到不同的队列上，每个队列可以由不同的CPU核心处理，从而利用多核优势。
   - 每个队列有自己的中断，可以通过设置中断亲和性（IRQ affinity）将中断绑定到特定CPU核心，以实现负载均衡和CPU缓存优化。
### 流程图概览
```
网卡接收数据 -> DMA到内存 -> 硬中断 -> 调度软中断 -> 软中断处理（NAPI轮询）-> 数据链路层解包 -> 网络层处理 -> 传输层处理 -> 放入socket接收缓冲区 -> 唤醒应用程序 -> 应用程序通过系统调用读取数据
```
### 性能优化点
   - **中断合并（Interrupt Coalescing）**：减少中断次数，通过设置在一段时间内或积累一定数量数据包后再触发中断。
   - **NAPI**：在高流量时，从中断模式切换为轮询模式，减少中断开销。
   - **RSS**：多队列网卡，并行处理。
   - **XDP（eXpress Data Path）**：在数据包进入内核协议栈前进行处理，可用于高性能场景（如DDOS防御、负载均衡）。
   - **零拷贝技术**：如`AF_XDP`、`DPDK`等，绕过内核协议栈，直接将数据包从网卡传递到用户空间。
理解这些步骤有助于进行网络性能调优和问题排查。


1. 物理层处理
信号接收：网卡（NIC）从物理线缆接收电信号/光信号。
数据帧解码：网卡将物理信号解码为二进制数据，形成以太网帧（Ethernet Frame）。

2. DMA 传输
DMA 写入内存：网卡通过 DMA（直接内存访问） 将数据帧直接写入内核预分配的 接收环形缓冲区（RX Ring Buffer），无需 CPU 参与。
缓冲区描述符更新：网卡更新接收描述符（Descriptor），通知内核有新数据到达。

3. 硬件中断
中断触发：网卡向 CPU 发送 硬件中断（IRQ），通知内核有数据包到达。
中断处理函数（上半部）：
	内核快速响应中断，调度 软中断（SoftIRQ） 处理后续任务。
	禁用网卡中断（避免中断风暴），后续通过轮询处理数据。

4. 软中断处理（下半部）
NET_RX_SOFTIRQ 软中断：由 ksoftirqd 内核线程处理，核心流程包括：
	NAPI 轮询（New API）：
		从 RX Ring Buffer 批量读取数据包（减少中断次数）。
		将原始数据包封装为 sk_buff 结构（内核网络数据包的核心结构）。
	数据链路层处理：
		检查以太网帧头（MAC 地址、类型字段）。
		剥离以太网帧头，根据协议类型（如 IPv4/IPv6）分发到网络层。

5. 网络层处理
IP 协议处理（ip_rcv() 函数）：
	验证 IP 头校验和。
	检查目的 IP 是否为本机（否则触发转发或丢弃）。
	根据 IP 协议号（如 TCP/UDP）分发到传输层。

6. 传输层处理
TCP/UDP 处理（以 TCP 为例，tcp_v4_rcv() 函数）：
	解析 TCP 头（端口号、序列号、标志位等）。
	根据 五元组（源IP、源端口、目的IP、目的端口、协议） 查找关联的 Socket。
	数据包排序、去重（处理乱序和重复包）。
	将数据存入 Socket 的 接收缓冲区（Receive Buffer）。

7. 唤醒应用程序
Socket 就绪通知：
	内核将数据放入 Socket 接收缓冲区后，标记 Socket 为可读。
	唤醒阻塞在 read()/recv() 系统调用上的应用进程。

8. 用户空间数据拷贝
系统调用处理：
	应用进程通过 recv() 或 read() 系统调用请求数据。
	内核将数据从 Socket 接收缓冲区拷贝到用户空间缓冲区。
	更新接收缓冲区状态（释放已读数据空间）。

关键优化技术
多队列网卡（RSS）：
	多数据流通过哈希分发到不同 CPU 核心，并行处理。

中断亲和性（IRQ Affinity）：
	将网卡中断绑定到特定 CPU 核心，减少缓存失效。

零拷贝技术：
	SO_ZEROCOPY：减少内核到用户空间的数据拷贝。
	AF_XDP：绕过内核协议栈，直接映射网卡数据到用户空间。

内核旁路：
	DPDK/Intel DPDK：用户态直接操作网卡，完全绕过内核。

完整流程图示
plaintext
 物理信号 → 网卡 → DMA → RX Ring Buffer → 硬中断 → 软中断(NAPI)
      ↓
  拆解以太网帧 → 网络层(IP) → 传输层(TCP/UDP) → Socket接收缓冲区
      ↓
   唤醒应用进程 ← recv()系统调用 ← 用户空间缓冲区

性能瓶颈点
中断处理延迟：硬件中断过多导致 CPU 负载高。
跨 NUMA 访问：网卡与 CPU 不在同一 NUMA 节点，内存访问延迟高。
单核处理瓶颈：单线程无法处理高速流量（需多队列/RSS）。
用户-内核态切换：频繁系统调用导致上下文切换开销。

💡 调优建议：
使用 ethtool 启用 RSS 和中断合并
绑定中断与进程到同一 NUMA 节点
启用 TCP 零拷贝（setsockopt(fd, SOL_SOCKET, SO_ZEROCOPY)）
增加 Socket 缓冲区大小（sysctl -w net.core.rmem_max=xxxx）

```

---
layout: article
tags: gitlab
title: irq suspension
mathjax: true
key: Linux
---

## irq suspension
```
gro-flush-timeout defer-hard-irqs irq-suspend-timeout可以在napi上设置，也可以全局设置·
from irq.sh
		$PERNAPI && {
			for ((q=0;q<$qcntcurr;q++)); do
				nid=${napimap[q]}
				sudo $NDCLI --do napi-set --json="{\"id\": $nid, \"gro-flush-timeout\": $2, \"defer-hard-irqs\": $3}" > /dev/null
				sudo $NDCLI --do napi-set --json="{\"id\": $nid, \"irq-suspend-timeout\": $4}" >/dev/null 2>&1 || echo "irq-suspend-timeout not available"
			done
		} || {
			sudo sh -c "echo $2 > /sys/class/net/$dev/gro_flush_timeout"
			sudo sh -c "echo $3 > /sys/class/net/$dev/napi_defer_hard_irqs"
#			[ -e /sys/class/net/$dev/irq_suspend_timeout ] && \
#				sudo sh -c "echo $4 > /sys/class/net/$dev/irq_suspend_timeout" || \
#				echo "irq_suspend_timeout not available"
	


busy_poll_usecs busy_poll_budget prefer_busy_poll传递给memcached了，并且最终传递给kernel：
+void epoll_set_kernelpoll(struct event_base* eb) {
+  int ret;
+  int epfd = eb->evbase->epfd;
+  struct epoll_params params = { 0, 0, 0 };
+  char* env = getenv("_MP_Usecs");
+  if (env) params.busy_poll_usecs = atoi(env);
+  env = getenv("_MP_Budget");
+  if (env) params.busy_poll_budget = atoi(env);
+  env = getenv("_MP_Prefer");
+  if (env) params.prefer_busy_poll = atoi(env);
+  ret = ioctl(epfd, EPIOCSPARAMS, &params);
+  assert(ret == 0);
+  (void)ret;
+}

参数	            类型	作用域	        约束条件
prefer_busy_poll    标志/选项	应用/文件描述符	触发忙轮询：告诉内核当前应用希望使用忙轮询。
busy_poll_usecs	    数值	系统全局	时间约束：限制忙轮询的最大持续时间。
busy_poll_budget    数值	系统全局	数量约束：限制忙轮询的最大处理数据量。


只有epoll返回了event，irq suspend就继续，irq_suspend_timeout是防止程序处理收到的数据都时间太长。

gro_flush_timeout should be set as small as possible, but large enough to
make sure that a single request is likely not being interfered with.

irq_suspend_timeout is largely a safety mechanism against misbehaving
applications. It should be set large enough to cover the processing of an
entire application batch, i.e., the factor between gro_flush_timeout and
irq_suspend_timeout should roughly correspond to the maximum batch size
that the target application would process in one go.

1. gro_flush_timeout (Baseline Time Unit)
This parameter defines the minimum time quantum related to IRQ deferral. It represents the time window over which small batches of packets (for GRO) are collected or, in this context, the granularity of basic IRQ handling when suspension is not active.

2. Maximum Batch Size (Multiplier)
This factor is dictated by the user application's logic. It is the maximum number of events (packets, frames, etc.) that the application intends to pull out of the network stack and process in a single, uninterrupted cycle after a successful epoll_wait call.

3. irq_suspend_timeout (The Safety Ceiling)
The resulting irq_suspend_timeout must be large enough to guarantee that the entire maximum batch size can be processed.

If the application processes the batch successfully and quickly, the actual suspension duration will be less than the timeout.

If the application takes longer than the timeout (perhaps due to a bug or excessive load), the timeout will expire, and IRQ handling will be forcefully resumed, preventing system lockup.

By linking the timeout to the application's internal batch size, the system ensures high-throughput, low-latency processing during peak load while maintaining a safety net against application misbehavior.



在 Linux 网络栈中，gro_flush_timeout 是一个专门用于优化 GRO (Generic Receive Offload) 机制性能的关键参数。它直接影响着数据包在进入协议栈之前的“聚合”效率和延迟。

为了理解这个参数，我们需要先明确 GRO 的背景。

1. 什么是 GRO？
GRO (通用接收卸载) 的核心目标是减少 CPU 的中断处理开销。
动作： 当网卡接收到大量属于同一个 TCP 流的小数据包时，GRO 会尝试将它们合并成一个大的“超长数据包”（Super-packet）。
结果： 内核协议栈只需要处理一次首部（Header），而不是处理几十次，从而显著提升吞吐量。

2. gro_flush_timeout 的定义
在传统的 GRO 中，合并通常是在一次 NAPI 轮询（Poll）循环内完成的。一旦轮询结束，合并包就会被立即推送到上层。
gro_flush_timeout 引入了一个时间窗口（单位通常为纳秒 ns）：
它允许内核在 NAPI 轮询结束后，不立即将合并了一半的数据包发给协议栈。
内核会等待一小段时间，期待在这段时间内还有属于同一流的更多数据包到达，从而实现更大规模的聚合。

3. 参数的工作原理
当一个 NAPI 实例（对应网卡的某个队列）收到了数据包：
合并阶段： 如果新到的包能和现有的合并，就进行合并。
等待阶段： 如果没有更多包了，但合并包还没达到最大长度（如 64KB），内核会检查 gro_flush_timeout。
决策：
如果设置了该超时，内核会启动一个定时器，暂时“扣留”这些包。
如果在超时之前有新包到达，继续合并。
如果超时时间到，强制“刷新”（Flush），将合并好的包送往上层。

4. 为什么这个参数很重要？
它是在 吞吐量 (Throughput) 与 延迟 (Latency) 之间做平衡的杠杆：

5. 与 NAPI 其它参数的配合
在高性能调优中，通常会看到它与 napi_defer_hard_irqs 配合使用：
napi_defer_hard_irqs：决定了如果没有新包，NAPI 轮询该坚持多久才重新开启硬件中断。
gro_flush_timeout：决定了在这个坚持的过程中，合并包最多能“攒”多久。
注意： 在你之前提到的 IRQ 悬挂 (IRQ Suspension) 提案中，gro_flush_timeout 被用作一个基础参考值。只有当 irq_suspend_timeout 设置得比它大时，更激进的“完全暂停中断”才会生效。

6. 如何查看和设置？
现代内核通常通过 netdev-genl 接口配置。如果是传统方式，可以通过 sysfs 查看（取决于驱动支持）：

# 查看某个网卡接口的 GRO 设置（示例）
cat /sys/class/net/eth0/gro_flush_timeout

总结
gro_flush_timeout 就像是快递站的“攒单”时间。快递员（内核）不是来一个件就送一次，而是等 10 分钟。如果这 10 分钟内还有同小区的件，就一起送。这样快递员效率最高，但第一件到的人会多等 10 分钟。


But the application must use the right APIs (NAPI/epoll config, epoll_wait, non-blocking sockets, well selected timeout values).

dnf -y install rpm-build
rpm -ivh kernel-6.12.0-174.el10.src.rpm 
cd rpmbuild/SPECS/
rpmbuild -bp kernel.spec 
cd ../BUILD/kernel-6.12.0-174.el10/linux-6.12.0-174.el10.x86_64/tools/net/ynl
make
#cd pyynl
#[root@dell-per740-91 pyynl]# ./ynl_gen_c.py --mode user --spec /root/rpmbuild/BUILD/kernel-6.12.0-174.el10/linux-6.12.0-174.el10.x86_64/Documentation/netlink/specs/netdev.yaml --header -o ../generated/netdev-user.h
#[root@dell-per740-91 pyynl]# ./ynl_gen_c.py --mode user --spec /root/rpmbuild/BUILD/kernel-6.12.0-174.el10/linux-6.12.0-174.el10.x86_64/Documentation/netlink/specs/netdev.yaml --source -o ../generated/netdev-user.c 

# suspendX
./irq_suspend_server -i 6 -s 20000000 -d 100 -r 50000 -u 100 -g 64 -P 1 -p 8000 -o received_data.bin

# deferX
./irq_suspend_server -i 6 -s 0 -d 100 -r 2000000 -p 8000 -o received_data.bin

# napibusy
./irq_suspend_server -i 6 -s 0 -d 100 -r 200000 -u 64 -g 64 -P 1 -p 8000 -o received_data.bin


# suspendX
./irq_suspend_server -i 6 -s 200000000 -d 100 -r 20000000 -u 64 -g 64 -P 1 -p 8000 -S suspend_stat

# deferX
./irq_suspend_server -i 6 -s 0 -d 100 -r 20000000 -p 8000 -S defer_stat

# napibusy
./irq_suspend_server -i 6 -s 0 -d 100 -r 20000000 -u 64 -g 64 -P 1 -p 8000 -S napibusy_stat


./test_client -s 199.111.1.1 -p 8000 -m 1024 -n 10000000
./test_client -s 199.111.1.1 -p 8000 -m 1024 -n 10000 -d 10000
./test_client -s 199.111.1.1 -p 8000 -m 1024 -n 10000000 -I -b 10000 -w 5 -D 10000
./test_client -s 199.111.1.1 -p 8000 -m 1024 -n 1000000 -I -b 10000 -w 1 -D 2000000 >/dev/null


```

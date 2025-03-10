---
layout: article
tags: Other
title: Interview
mathjax: true
key: Linux
---

## Python
```
1. Do you know python comprehensions? list comprehensions
https://www.runoob.com/python3/python-comprehensions.html
[表达式 for 变量 in 列表] 
[out_exp_res for out_exp in input_list]


2. Do you know python closure? https://github.com/Jwindler/Ice_story/blob/main/src/python/closure.md
闭包（closure）是一个函数对象，它与它的环境变量（包括自由变量）的引用组合而成的实体。
闭包可以保留函数定义时所在的环境变量，即使这些变量在定义时不在该函数的作用域内也可以使用。闭包可以让函数像对象一样被传递、赋值、作为参数传递，甚至在函数内部定义函数。

3. Python decorator.
https://l8liliang.github.io/2024/04/25/decorator.html

4. How to traverse python dictionary? 

5. What's difference between python list and tuple and set?

6. How to remove the duplicated item of a python list?

7. What python module did you ever use?

8. regular expression https://www.runoob.com/python3/python3-reg-expressions.html
differenct between re.match re.search
re.match 只匹配字符串的开始，如果字符串开始不符合正则表达式，则匹配失败，函数返回 None，而 re.search 匹配整个字符串，直到找到一个匹配。

9. substitute: Python 的re模块提供了re.sub用于替换字符串中的匹配项。

10. summarize the different way to import a python module
https://l8liliang.github.io/2024/03/05/python-import.html
import sys //import 整个模块(需要使用前缀)
import sys as s //设置别名(需要使用前缀)
from sys import argv //无须使用任何前缀

11. python函数参数数量不固定
在Python中，可以使用*args和**kwargs来定义可接收不定数量的参数的函数。
*args是用来发送一个非键值对的可变参数列表给函数，而**kwargs允许你将不定长度的键值对，作为参数传递给一个函数。


def fun_args(*args):
    for i in args:
        print(i)
 
fun_args(1, 2, 3, 4)

def fun_kwargs(**kwargs):
    for key, value in kwargs.items():
        print(f"{key}: {value}")
 
fun_kwargs(a=1, b=2, c=3)
```

## shell
```
1. talk about sed,  what's it used for?

# 移除空行
sed '/^$/d' file

# 替换每行中模式首次匹配的内容
sed 's/pattern/replace_string/' file

# 使用g标记执行全局替换
sed 's/pattern/replace_string/g' file

2. How to show current opening port?
netstat
lsof

3. Talk about linux signal?
https://l8liliang.github.io/2020/04/11/Linux_Schell_Scripting_Cookbook-10.html

4. find
find /home/slynux -name '*.txt' -print  // 在/home/slynux查找txt文件。

find . -type f -size +2k
find . -type f -size -2k
find . -type f -size 2k

5. source与./ point slash
source : 是在当前进程执行脚本
./     : 是在子进程执行脚本

6. 数组长度
echo ${#array_var[*]}
# 创建并初始化数组
array_name=(element1 element2 element3 ...)
# 访问数组元素
echo ${array_name[index]}
# 获取数组所有元素
echo ${array_name[@]}
# 获取数组长度
echo ${#array_name[@]}

7. The differenct between one square brackets and double square brackets
https://l8liliang.github.io/2023/11/30/shell_bracket.html

8. getopts
https://l8liliang.github.io/2023/07/20/getopt.html
```

## git
```
1. How to drop workspace changing?
git checkout – file

2. How to revert a file to a previous verison?
git reset --hard HEAD^

3. How to create a branch and switch to it with a command?
创建+切换分支：git checkout -b name

4. Hwo to know who last time changed a line in a file? 
git blame -L 1,10 nic_info


```

## upstream process
```
Did you ever contribute to linux upstream?
```

## network
```
1. ipv6 address type
https://l8liliang.github.io/2022/01/04/ipv6-address.html

2. icmpv6 https://l8liliang.github.io/2022/01/05/icmpv6.html
路由器和主机之间发送的，行使动态地址分配功能的消息：
1.路由器恳求消息RS Router Solicitation
2.路由器通告消息RA Router Advertisement

主机之间，行使地址解析功能的消息：
3.邻居恳求消息NS Neighbor Solicitation
3.邻居通告消息NA Neighbor Advertisement

```

## pci
```
1. How to check pci bandwidth
https://docs.google.com/presentation/d/19R8VZPFN1T3Ap9FrytfDPbMKLMpz_38n33e4cf3kGpw/edit#slide=id.gcf7cf741d2_0_26
use lapci
Every PCIe device has a BDF(Bus,Device,Function)

Driver register to PCI subsystem by calling pci_register_driver()
PCI subsystem scan device and find driver by device id and call driver probe()
Driver control HW by calling functions provided by PCI subsystem
Every PCIe device has a BDF(Bus,Device,Function)
Data transfer via PCIe bus

2. Can you describe PCIe Bus Struct
Root Complex : RC is used to connect CPU and PCIe.
pcie switch 
pice bridge
pcie endpoint

3. PCIe Layers
https://docs.google.com/presentation/d/19R8VZPFN1T3Ap9FrytfDPbMKLMpz_38n33e4cf3kGpw/edit#slide=id.gcf7cf741d2_0_14

4. How to list pci tree in linux system.
lspci -t
```

## network performance
```
How to tunning network performance?
Try to list all the linux configuration item relevant to network performance.

https://l8liliang.github.io/2021/08/19/linux-network-performance.html

• SoftIRQ misses (netdev budget)
• "tuned" tuning daemon
• "numad" NUMA daemon
• CPU power states
• Interrupt balancing 
• Pause frames
• Interrupt Coalescence
• Adapter queue (netdev backlog)
• Adapter RX and TX buffers
• Adapter TX queue
• Module parameters
• Adapter offloading
• Jumbo Frames
• TCP and UDP protocol tuning
• NUMA locality

1. what's NUMA
https://l8liliang.github.io/2021/08/19/linux-network-performance.html#numa

2. Can bind a port to another specific NUMB
echo 0 > /sys/class/net/ens1f0/device/numa_node 

3. bind process to NUMA
使用numactl把进程绑定到网卡所属numa
numactl --cpubind=netdev:ens4f1 --membind=netdev:ens4f1 iperf3 -s -D -p $port -B 199.3.3.1 &> log$port &

4. ira balance and irq affinity
echo $affinity > /proc/irq/$irq/smp_affinity
```

## The underlying technique of sriov, why sriov vnic has same throughput with physical NIC?
```
iommu
```

## NIC offload
```
TSO tcp
UFO udp
GSO tcp/udp

LRO tcp
GRO tcp/udp

How to check if TSO/GRO offload is working well?
```

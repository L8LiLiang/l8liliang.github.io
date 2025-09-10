---
layout: article
tags: Linux
title: SYNCE & DPLL
mathjax: true
key: Linux
---

[Reference1](https://www.h3c.com/cn/Service/Document_Software/Document_Center/Home/Switches/00-Public/Learn_Technologies/White_Paper/(SyncE)_WP-6W100/)
{:.info}

[Reference2](https://www.embedded.com/an-introduction-to-synchronized-ethernet/)
{:.info}

[Reference3](https://www.cisco.com/c/en/us/support/docs/ios-nx-os-software/ios-xr-software/217579-configure-ptp-and-synce-basics-with-cisc.html#anc7)
{:.info}

[Reference4](https://www.ciscopress.com/articles/article.asp?p=3128857&seqNum=2)
{:.info}

## Clock
```
In everyday usage, the term clock refers to a device that maintains and displays the time of day and perhaps the date. 
In the world of electronics, however, clock refers to a microchip that generates a clock signal, 
which is used to regulate the timing and speed of the components on a circuit board. 
This clock signal is a waveform that is generated either by a clock generator or the clock itself—the most common form of clock signal in electronics is a square wave.
```

## PLL
```
A phase-locked loop (PLL) is an electronic device or circuit that generates an output clock signal that 
is phase-aligned as well as frequency-aligned to an input clock signal. 
As shown in Figure 5-3, in its simplest form, a PLL circuit consists of three basic elements, as described in the list that follows.
```

## Low-Pass and High-Pass Filters
```
A low-pass filter (LPF) is a filter that filters out (removes) signals that are higher than a fixed frequency, 
which means an LPF passes only signals that are lower than a certain frequency—hence the name low-pass filter. 
For the same reasons, sometimes LPFs are also called high-cut filters because they cut off signals higher than some fixed frequency. 
Figure 5-5 illustrates LPF filtering where signals with lower frequency than a cut-off frequency are not attenuated (diminished). 
The pass band is the range of frequencies that are not attenuated, and the stop band is the range of frequencies that are attenuated.

When the PLL is tracking an input reference, it is the loop filter that is enforcing the limits of these frequency ranges. 
The loop filter is usually a low-pass filter, and as the name suggests, it allows only low-frequency (slow) variations to pass through. 
That means it removes high-frequency variation and noise in the reference signal. Conversely, a high-pass filter allows high-frequency variations to pass and removes the low-frequency changes.

While these filters are discussed in the next section in detail, it is important to note that “low-frequency variations” does not refer to the frequency of the clock signal, 
but the rate with which the frequency or phase of the clock signal varies. 
Low rate (less frequent) changes are a gradual wander in the signal, whereas high rate changes are a very short-term jitter in the signal. 
You will read more about jitter and wander later in this chapter.
```

## Jitter and Wander
```
Noise is classified as either jitter or wander. As seen in the preceding section “Jitter and Wander,” by convention, 
jitter is the short-term variation, and wander is the long-term variation of the measured clock compared to the reference clock. 
The ITU-T G.810 standard defines jitter as phase variation with a rate of change greater than or equal to 10 Hz, 
whereas it defines wander as phase variation with a rate of change less than 10 Hz. In other words, slower-moving, low-frequency jitter (less than 10 Hz) is called wander.

https://www.ciscopress.com/articles/article.asp?p=3128857
如这里面的Figure5-8: jitter表示每个上升沿或者下降沿的时间差别，wander表示时间差别在变大还是缩小。

Jitter和Wander就在哪里，你可以用不同的方法去测量。
比如，MTIE和TDEV都属于一种Wander
```

## Frequency Error
```
While jitter and wander are both metrics to measure phase errors, the frequency error (or accuracy) also needs to be measured.

The frequency error (also referred to as frequency accuracy) is the degree to which the frequency of a clock can deviate from a nominal (or reference) frequency. 
The metric to measure this degree is called the fractional frequency deviation (FFD) or sometimes just the frequency offset. 
This offset is also referred to as the fractional frequency offset (FFO).

y(t)=(v(t)-v(nom))/v(nom)

where:

y(t) is the FFD at time t
v(t) is the frequency being measured and
v(nom) is the nominal (or reference) frequency

FFD is often expressed in parts per million (ppm) or parts per billion (ppb).

```

## Time Error
```
就是两个时钟的时间差,相位差
```

## max TE
```
max|TE|, which is defined as the maximum absolute value of the time error observed over the course of the measurement
```

## Time Interval Error
```
see https://www.ciscopress.com/articles/article.asp?p=3128857&seqNum=2

The time interval error (TIE) is the measure of the change in the TE over an observation interval. 

In comparison, the TE is the instantaneous measure between the two clocks; there is no interval being observed. 
So, while the TE is the recorded error between the two clocks at any one instance, the TIE is the error accumulated over the interval (length) of an observation. 
Another way to think of it is that the TE is the relative time error between two clocks at a point (instant) of time, 
whereas TIE is the relative time error between two clocks accumulated between two points of time (which is an interval).

TE是绝对error，TIE是intervel内积累的新error.


一般测试的时候，应该是随着测试时间增加，interval也增加，就是说interval的开始时间是0，结束时间是当前的测试运行的时间：
Some conclusions that you can draw based on plotted TIE on a graph are as follows:

An ever-increasing trend in the TIE graph suggests that the measured clock has a frequency offset (compared to the reference clock). You can infer this because the frequency offset will be seen in each time error value and hence will get reflected in TIE calculations.

If TIE graph shows a large value at the start of the measurement interval and starts converging slowly toward zero, it might suggest that the clock or PLL is not yet locked to the reference clock or is slow in locking or responding to the reference input.

TIE适合测量jitter:
Note that the measurement of the change in TE at each interval really becomes measurement of a short-term variation, or jitter, in the clock signals, or what is sometimes called timing jitter. 
And as TIE is a measurement of the change in phase, it becomes a perfect measurement for capturing jitter.

accuracy and stability:
Now it is time to revisit the concepts of accuracy and stability from Chapter 1, “Introduction to Synchronization and Timing.” 
These are two terms that have a very precise meaning in the context of timing.

For synchronization, accuracy measures how close one clock is to the reference clock. 
And this measurement is related to the maximum absolute TE (or max|TE|). So, a clock that closely follows the reference clock with a very small offset is an accurate clock.

On the other hand, stability refers to the change and the speed of change in the clock during a given observation interval, 
while saying nothing about how close it is to a reference clock.

```

## Constant Versus Dynamic Time Error
```
Constant time error (cTE) is the mean of the TE values that have been measured. 
The TE mean is calculated by averaging measurements over either some fixed time period (say 1000 seconds) or the whole measurement period.

Dynamic time error (dTE) is the variation of TE over a certain time interval (you may remember, the variation of TE is also measured by TIE).
Additionally, the variation of TE over a longer time period is known as wander, so the dTE is effectively a representation of wander.
Another way to think about it is that dTE is a measure of the stability of the clock.

These two metrics are very commonly used to define timing performance, so they are important concepts to understand. 
Normally, dTE is further statistically analyzed using MTIE and TDEV; and the next section details the derivations of those two metrics.

cTE用来测量accurancy，dTE用来测量stability,MTIE和TDEV都属于dTE,and the dTE is effectively a representation of wander, 所以MTIE和TDEV都测量wander?

```

## Maximum Time Interval Error
```
TIE是取x(n+k)-x(n) , 这里的x(n)指的是在时钟周期n时候测量出的TE
但是这两个指（x(n+k)， x(n)）不一定是最大和最小值。
MTIE是找到这个interval之间的最大值和最小值，取差。

Note also that as the MTIE is the maximum value of delay variation, it is recorded as a real maximum value; 
not only for one observation interval but for all observation intervals. 
This means that if a higher value of MTIE is measured for subsequent observation intervals, it is recorded as a new maximum or else the MTIE value remains the same. 
This in turn means that the MTIE values never decrease over longer and longer observation intervals.

Therefore, MTIE can only stay the same or increase if another higher value is found during the measurement using a longer observation interval; 
hence, the MTIE graph will never decline as it moves to the right (with increasing tau).

The MTIE graph remaining flat means that the maximum peak-to-peak delay variation remains constant; 
or in other words, no new maximum delay variation was recorded. On the other hand, if the graph increases over time, 
it suggests that the test equipment is recording ever-higher maximum peak-to-peak delay variations. 
Also, if the graph resembles a line increasing linearly, it shows that the measured clock is not locked to the reference clock at all, 
and continually wandering off without correction. MTIE is often used to define a limit on the maximum phase deviation of a clock signal.

To summarize, MTIE values record the peak-to-peak phase variations or fluctuations, 
and these peak-to-peak phase variations can point to frequency shift in the clock signal. 
Thus, MTIE is very useful in identifying a shift in frequency.
```

## Time Deviation
```
Whereas MTIE shows the largest phase swings for various observation intervals, time deviation (TDEV) provides information about phase stability of a clock signal. 
TDEV is a metric to measure and characterize the degree of phase variations present in the clock signal, primarily calculated from TIE measurements.
TDEV测量相位变化幅度

Unlike MTIE, which records the difference between the high and low peaks of phase variations, 
TDEV primarily focuses on how frequent and how stable (or unstable) such phase variations are occurring over a given time—note the importance of “over a given time.” 


# from gemini
This text provides a great description of Time Deviation (TDEV), a key metric for understanding clock performance.

In simple terms, while MTIE tells you the worst-case error in a clock's time, 
TDEV tells you about the clock's frequency stability—that is, how "smoothly" or "consistently" it ticks over different time scales.

An Analogy: A Long Car Trip
Imagine you are on a long road trip with a GPS navigator showing the perfect, planned route.

MTIE (Maximum Time Interval Error): This is like asking, "During any 10-minute segment of our trip, what was the farthest we ever strayed from the planned route?" 
It only cares about the single worst-case deviation in position. You might have driven perfectly smoothly but took one wrong turn, and the MTIE would be very high.

TDEV (Time Deviation): This is like asking, "On average, over 10-minute segments, how much did our speed fluctuate?" 
TDEV doesn't care as much about your maximum distance from the route. 
It cares about whether you were constantly speeding up and slowing down (high TDEV) or if you maintained a very steady speed (low TDEV). 
A low TDEV indicates a smooth, stable journey.

MTIE指出最大时间偏差，
TDEV指出频率的稳定性。

What the Description Means
"TDEV provides information about phase stability" vs. "MTIE shows the largest phase swings"
This is the core difference. 
MTIE is a peak-to-peak measurement, capturing the worst-case boundary of the time error. 
TDEV is a statistical root-mean-square (RMS) measurement that characterizes the stability of the clock's average frequency.

"TDEV primarily focuses on how frequent and how stable...such phase variations are"
This highlights TDEV's statistical nature. 
It's not just about a single bad event; it's about the overall pattern of instability. 
A clock that consistently has small errors will have a better TDEV than a clock with fewer but more erratic errors.

"Think of tossing five coins..."
This is a perfect analogy for what TDEV does.

Getting five heads on the first toss (MTIE might be high) looks alarming, but it could be a random fluke.

TDEV is like tossing the coins hundreds of times. 
By analyzing the results, you can determine with high confidence if the coins are fair (the clock is stable and its variations are predictable random noise) or 
if they are biased (the clock has a systematic problem, like a consistent frequency drift).

Why TDEV is Useful
TDEV is an essential tool for engineers because it helps them understand the type of noise affecting an oscillator. 
Different types of noise (like white phase noise, flicker phase noise, random walk frequency noise) show up as distinct signatures on a TDEV plot.

In summary, if you need to know the maximum time error to size a network buffer, you use MTIE. 
If you need to understand the intrinsic stability and quality of the clock's oscillator, you use TDEV.
```

## Noise
```
The metrics that are discussed in this chapter try to quantify the noise that a clock can generate at its output. 
For example, max|TE| represents the maximum amount of noise that can be generated by a clock. Similarly, cTE and dTE also characterize different aspects of the noise of a clock.

Noise is classified as either jitter or wander. As seen in the preceding section “Jitter and Wander,” by convention, jitter is the short-term variation, 
and wander is the long-term variation of the measured clock compared to the reference clock. 
The ITU-T G.810 standard defines jitter as phase variation with a rate of change greater than or equal to 10 Hz, 
whereas it defines wander as phase variation with a rate of change less than 10 Hz. In other words, slower-moving, low-frequency jitter (less than 10 Hz) is called wander.


The preceding sections explained that MTIE and TDEV are metrics that can be used to quantify the timing characteristics of clocks. 
Therefore, it is no surprise that the timing characteristics of these clocks based on the noise performance (specifically noise generation) 
have been specified by ITU-T in terms of MTIE and TDEV masks. These definitions are spread across numerous ITU-T specifications, 
such as G.811, G.812, G.813, and G.8262. Chapter 8 covers these specifications in more detail.

Because these metrics are so important, any timing test equipment used to measure the quality of clocks needs to be able to generate MTIE and TDEV graphs as output. 
The results on these graphs are then compared to one of the masks from the relevant ITU-T recommendation. 
However, it is imperative that the engineer uses the correct mask to compare the measured results against.

# Noise Generation
Noise generation is the amount of jitter and wander that is added to a perfect reference signal at the output of the clock.

# Noise Tolerance
A slave clock can lock to the input signal from a reference master clock, and yet every clock (even reference clocks) generates some additional noise on its output. 
As described in the previous section, there are metrics that quantify the noise that is generated by a clock. 
But how much noise can a slave clock receive (tolerate) on its input and still be able to maintain its output signal within the prescribed performance limits?
It is simply a measure of how bad an input signal can become before the clock can no longer use it as a reference.

# Noise Transfer
This propagation of noise from input to output of a node is called noise transfer and can be defined as the noise that is seen at the output 
of the clock due to the noise that is fed to the input of the clock. 
The filtering capabilities of the clock determines the amount of noise being transferred through a clock from the input to the output. Obviously, less noise transfer is desirable
```

## Transient Response
```
The question arises, how should the clock behave when an input reference clock is lost and the clock decides to select the second-best clock source as the input reference? 
The behavior during this event is characterized as the transient response of a clock.
```

## 术语
```
同步以太网（SyncE，Synchronous Ethernet）
Primary Reference Clock (PRC)
Primary Reference Source (PRS)
Synchronization Supply Unit (SSU)
Building Integrated Timing Supply (BITS)
SDH Equipment Clock (SEC)
SONET Minimum Clock (SMC)
Digital Phase Locked Loop(DPLL)
```

## 产生背景
```
在通信网络中，许多业务的正常运行都要求网络时间同步。时间同步包括频率和相位两方面的同步。通过时间同步可以使整个网络各设备之间的频率和相位差保持在合理的误差范围内。
同步以太网（SyncE，Synchronous Ethernet）是一种基于物理层码流携带和恢复频率信息的同步技术，能实现网络设备间高精度的频率同步，满足无线接入业务对频率同步的要求。
通常将SyncE和PTP技术配合使用，同时满足频率和相位的高精度要求，实现纳秒级的时间同步。
```

## 时间同步
```
频率同步也称为时钟同步。频率同步指两个信号的变化频率相同或保持固定的比例，信号之间保持恒定的相位差。
相位同步是指信号之间的频率和相位都保持一致，即信号之间相位差恒定为零。

SyncE只能实现频率同步，SyncE和PTP配合可实现时间同步。
```

## 时钟源类型
```
为设备提供时钟信号的设备叫做时钟源。根据时钟信号的来源不同，SyncE支持的时钟源包括：

·     BITS（Building Integrated Timing Supply System，通信楼综合定时供给系统）时钟源：时钟信号由专门的BITS时钟设备产生。设备通过专用接口（BITS接口）收发BITS时钟信号。
·     线路时钟源：由上游设备提供的、本设备的时钟监控模块从以太线路码流中提取的时钟信号，即开启SyncE功能的接口传递的时钟信号。线路时钟源精度比BITS时钟源低。
·     PTP时钟源：本设备从PTP协议报文中提取的时钟信号。PTP协议时钟源的精度比BITS时钟源低。
·     本地时钟源：本设备内部的晶体振荡器产生的38.88 MHz时钟信号，通常本地时钟源精度最低。
```

## 时钟源选择
```
当设备连接了多种时钟源，有多路时钟信号输入设备时，可以通过手动模式或自动模式选择一路优先级最高的时钟信号作为最优时钟（也称为参考源）。
·     手动模式：用户手工指定最优时钟。如果最优时钟的同步信号丢失，设备不会自动采用其他时钟源的时钟信号，而是使用设备上存储的已丢失的最优时钟的时钟参数继续运行。
·     自动模式：系统自动选择最优时钟（也称为自动选源）。如果最优时钟的同步信号丢失，设备会自动选择新的最优时钟，并和新的最优时钟保持同步。

自动选源参考因素
影响设备自动选择最优时钟的参考因素包括SSM（Synchronization Status Message，同步状态信息）级别和优先级。

1. SSM级别
SSM是ITU-T G.781在SDH（Synchronous Digital Hierarchy，同步数字系列）网络中定义的标识时钟源质量等级（QL，Quality Level）的一组状态信息。
SyncE也使用SSM级别来表示时钟源的好坏，并把SSM级别称为QL级别，本文中统称为SSM级别。

SyncE中支持的SSM级别按照其同步质量由高到低依次为：
·     PRC（Primary Reference Clock，基准参考时钟）级别：精度符合G.811协议要求。通常，BITS跟踪GPS或者北斗等卫星源后输出的时钟信号的质量等级为PRC。
·     SSU-A（primary level SSU，转接局时钟）级别：精度符合G.812中转接节点要求。通常，配置了铷钟的BITS在丢失GPS等卫星源后进入保持或者自由振荡时输出的时钟信号的质量等级为SSU-A。
·     SSU-B（second level SSU，本地局时钟）级别：精度符合G.812中本地节点要求。通常，配置了晶体钟的BITS在丢失GPS等卫星源后进入保持或者自由振荡时输出的时钟信号的质量等级为SSU-B。
·     SEC（SDH Equipment Clock，SDH设备时钟）/EEC（Ethernet Equipment Clock，以太网设备时钟）级别：精度符合G.813协议要求。
              通常，承载网设备（SDH设备SEC或者同步以太设备EEC）丢失参考源后进入保持或者自由振荡时输出的时钟信号的质量等级为SEC/EEC。
·     DNU（Do Not Use for synchronization，不应用作同步）级别：精度不符合时钟同步的要求。该质量等级的时钟源不可以作为参考源。
·     UNK级别：同步质量未知。


SSM级别对于自动选择最优时钟、时钟环路避免具有重要意义。设备间通过周期发送ESMC（Ethernet Synchronous Message Channel，以太网同步消息通道）报文传递时钟源的SSM级别。
ESMC报文有两种发送方式：
·     周期发送：设备在开启了SyncE功能的接口上每秒发送一次ESMC information报文，用于告知邻居设备本设备提供的时钟信号的SSM级别。
·     事件触发发送：当本设备选择的最优时钟变化时，立即发送携带新的最优时钟的SSM级别的ESMC event报文，以便尽快通知下游设备本设备提供的时钟信号的SSM级别发生变换。
                    与此同时，复位ESMC information报文的发送定时器，并周期性发送携带新SSM级别的ESMC information报文。

自动选源机制
(1)     SSM级别最高的时钟源优先当选为最优时钟。
(2)     如果用户配置了SSM级别不参与自动选源，或者SSM级别相同，则按照时钟源的优先级进行选择，优先级值最小的时钟源优先被选中。
(3)     如果时钟源的优先级相同，则按照时钟源类型进行选择，优先选用BITS时钟源，其次选用线路时钟源，然后选用PTP时钟源。
(4)     如果时钟源的类型也相同，继续比较时钟信号入接口的编号，编号最小的时钟源优先被选中。
(5)     当BITS时钟源、线路时钟源、PTP时钟源均不可用时，使用本地时钟源。

选举出最优时钟后，设备会通过ESMC报文将最优时钟的SSM级别传递给下游设备，进一步影响下游设备最优时钟的选择。

如果某接口收到的时钟信号当选为最优时钟，而该接口在5秒钟内未收到ESMC information报文，设备会认为最优时钟丢失或不可用，将自动按照上述原则重新选择最优时钟。
当原最优时钟源恢复时，系统自动立即切换回原最优时钟。
```

## 时钟同步原理
```
选出最优时钟后，设备开始锁定最优时钟，进行时钟同步（频率同步）。

数字通信网中传递的是对信息进行编码后得到的PCM（Pulse Code Modulation，脉冲编码调制）数字脉冲信号，每秒生成的脉冲个数即为脉冲的频率。
以太网物理层编码采用FE（百兆）和GE（千兆）技术，平均每4个比特就插入一个附加比特，这样在其所传输的数据码流中不会出现超过4个1或者4个0的连续码流，可有效地包含时钟信息。
利用这种信息传输机制，SyncE在以太网源端接口上使用高精度的时钟发送数据，在接收端恢复、提取这个时钟，并作为接收端发送数据码流的基准。
In fact, the old 10Mbps (10Base-T) Ethernet is not even capable of synchronization signal transmission over the physical layer interface because a 10Base-T transmitter stops sending pulses during idle periods.

A 10Base-T transmitter simply sends a single pulse (“I am alive” pulse) every 16 ms to notify its presence to the receiving end. Of course, such infrequent pulses are not sufficient for clock recovery at the receiver.

Idle periods in faster Ethernet flavors (100Mbps, 1Gbps and 10Gbps) are continuously filed with pulse transitions, allowing continuous high-quality clock recovery at the receiver–good candidates for synchronized Ethernet.


假设外接时钟源1比外接时钟源2更可靠，当选为最优时钟。Device1和Device2均同步外接时钟源1的频率，同步原理如下：

1. 发送方向同步机制
发送端携带并传递同步信息：
(1)     因为外接时钟源1的SSM级别最高，Device1选择外接时钟源1作为最优时钟。
(2)     Device1提取外接时钟源1发送的时钟信号，并将时钟信号注入以太网接口卡的PHY芯片中。
(3)     PHY芯片将这个高精度的时钟信息添加在以太网线路的串行码流里发送出去，向下游设备Device2传递时钟信息。

2. 接收方向同步机制
接收方向提取并同步时钟信息：
(1)     Device2的以太网接口卡PHY芯片从以太网线路收到的串行码流里提取发送端的时钟信息，分频之后上送到时钟扣板。
(2)     时钟扣板将接口接收的线路时钟信号、外接时钟源2输入的时钟信号、本地晶振产生的时钟信号进行比较，根据自动选源算法选举出线路时钟信号作为最优时钟，并将时钟信号发送给时钟扣板上的锁相PLL。
(3)     PLL跟踪时钟参考源后，同步本地系统时钟，并将本地系统时钟注入以太网接口卡PHY芯片往下游继续发送，同时将本地系统时钟输出给本设备的业务模块使用。
```

## 时钟工作状态
```
系统时钟存在三种工作状态，用户通过查看系统时钟的状态信息，可了解设备的时钟同步情况：
·     跟踪状态：当设备选择了一个最优时钟，并和最优时钟达到频率同步，将处于跟踪状态。系统时钟处于跟踪状态时，时钟芯片内部会不断保存最优时钟的相关数据。
·     保持状态：当最优时钟失效，不能继续提供时钟信号时，时钟芯片会根据之前存储的相关数据，在一定时间（最长不超过24小时）内保持之前最优时钟的频率特征，提供与原最优时钟相符的时钟信号。此时，系统时钟处于保持状态。
·     自由振荡状态：若保持状态超时，原最优时钟仍未恢复，系统时钟会进入自由振荡状态。此时，设备使用内部晶振作为最优时钟。
```

## dpll
```
Digital Phase Locked Loop(DPLL)

+--------------------------+     +-------------------------------------------------+
|device1                   |     |device2                                          |
|                          |     |                                                 |
|+------------+    +-----+ |     | +-----+                                         |
|| clock      | -> | phy | | --> | | phy |<---------------+------------>业务模块   |
|+------------+    +-----+ |     | +--+--+                |系统时钟                |
+--------------------------+     |    |recovery clock     ｜                       |
                                 |    +------------------->dpll                    |
                                 +-------------------------------------------------+

接收方向提取并同步时钟信息：
(1)     Device2的以太网接口卡PHY芯片从以太网线路收到的串行码流里提取发送端的时钟信息，分频之后上送到时钟扣板。
(2)     时钟扣板将接口接收的线路时钟信号、外接时钟源2输入的时钟信号、本地晶振产生的时钟信号进行比较，根据自动选源算法选举出线路时钟信号作为最优时钟，并将时钟信号发送给时钟扣板上的锁相PLL。
(3)     PLL跟踪时钟参考源后，同步本地系统时钟，并将本地系统时钟注入以太网接口卡PHY芯片往下游继续发送，同时将本地系统时钟输出给本设备的业务模块使用。


Any Gigabit or 10 Gigabit Ethernet PHY device should be able to support synchronized Ethernet, so long as it provides a recovered clock on one of its output pins. 
The recovered clock is cleaned by the PLL and fed to the 25MHz crystal oscillator input pin on the PHY device. 
Some new Ethernet PHY devices provide a dedicated pin for the synchronization input. 
The advantage of this approach is that frequency input can be higher than 25MHz–higher clock frequencies usually have lower jitter. 
In addition, this approach avoids any potential timing loop problems within the PHY device.

From the discussion so far, it appears that the only requirement for a PLL used in SyncE is to clean jitter from the recovered clock, which can be accomplished with general purpose PLLs. 
However, the PLL used in SyncE must provide additional functions beyond jitter cleaning.

Free-run accuracy: 
The accuracy of PLL output when it is not driven by a reference should be equal or better than +/-4.6 ppm (part per million) over a time period of one year. 
This is a very accurate clock relative to the clock accuracy for traditional Ethernet (+/-100 ppm).

Holdover: 
The PLL constantly calculates the average frequency of the locked reference. If the reference fails and no other references are available, 
the PLL goes into holdover mode and generates an output clock based on a calculated average value. 
Holdover stability depends on the resolution of the PLL averaging algorithm and the frequency stability of the oscillator used as the PLL master clock.

Reference monitoring: 
The PLL needs to constantly monitor the quality of its input references. If the reference deteriorates (disappears or drifts in frequency), 
then the PLL raises an alarm (interrupt) and switches to another valid reference.

Hitless reference switching: 
If the PLL's reference fails, then it will lock to another available reference without phase disturbances at its output.

Jitter and wander filtering: 
The PLL can be viewed as a jitter and wander filter. The narrower the loop bandwidth, the better the jitter and wander attenuation.

Jitter and wander tolerance: 
The PLL should tolerate large jitter and wander at its input and still maintain synchronization without raising any alarms.
```

## E810

### eEEC and 1PPS
```
> Hello Pasi,
>
> Do you know why DPLL outputs two signals, i.e. DPLL0 outputting a
> clock signal and DPLL1 outputting 1PPS signal?
>
DPLL0只处理频率，来源包括RCLK和PTP，(当只有GNSS 1PPS输入的时候，也通过1PPS进行频率同步）。
DPLL1处理相位，来源包括GNSS和PTP。
Yes, DPLL0 is eEEC and deals with the frequency only (including input
sources other than GNSS, such as clocks recovered from Ethernet
interfaces or E810 when PTP is acting as source), while DPLL1 deals with
the phase only (i.e. it is syncronized to 1Hz pulse per second,
irrespective of the source), which is basically "exact" (to the extent
possible) location of the 1Second boundary, irrespective of it's source
(either GNSS receiver or PTP / E810).
> From my understanding the clock signal is for synchronizing frequency 
> and 1PPS signal is for synchronizing phase?
>
yes, see above
> But with 1PPS input from GNSS, DPLL can achieve both frequency and
> phase synchronization.
>
GNSS其实也输出频率，但是没有连接到DPLL。
如果只连接了1PPS，DPLL0和DPLL1都使用1PPS进行同步。
Yes, it can be debated whether this is optimal, but it works well enough
in this case. There is also freq output from GNSS module but it is not
physically wired to DPLL, so both frequency and phase aligment pieces
are sourced from 1PPS signal only. Both DPLLs in this case use 1PPS as
inputs.
> So it seems 1PPS is enough for frequency and phase synchronization,
> why DPLL outputs two signals?
>
没太看懂，DPLL会输出多个phase和frequency？
DPLL outputs way more than two signals on both phase (although in that
case they are generally all 1PPS phase aligned signals irrespective of
the destination). On frequency side there are many outputs as well, and
with different frequencies (10Mhz, 155+ Mhz, clocik to E810 etc.).
> What's the difference between clock signal and 1PPS signal?
clock就是固定频率的信号，不能表示ToD.
Clock is what it sounds like, a periodic signal with specific frequency
(e.g. 10Mhz clock has 10M transitions per second), clock signal can not
be signify time by itself, but high speed clocks are required for
various purposes, including ns level timekeeping as well as PHY clocks
etc. Clock signal transitions are not necessarily aligned in phase with
the 1PPS transition, although SOME of them are or can be, it dpends on
the specific clock and it's purpose.

ToD is ASCII string from GNSS module in GM case, (in WPC/LB case it's NMEA sentence) which tells what second the 1PPS lo-hi transition edge is associated with, 
as from the transition alone you only  know that it is start of new second, which is necessary but not sufficient 
to figure out the time which is what we are fundamentally trying to sychronize w/ 1PPSs.

Pasi
```

### E810 Architecture
```
(output) U.FL1   U.FL2(input)    GNSS
	   ^       |              |
           |       |              |
           +       |              v
SMA1----SMA Logic<-|--------->DPLL(        ) -----------------+---+--+--+--+
                   |           ^  ^  ^   |R                   |   ^  |  ^  |REFCLK
              +----+           |  |R |R  |E                   |S  |S |S |S |
              |                |  |C |C  |F                   |D  |D |D |D |
              v                |  |L |L  |C                   |P  |P |P |P |
SMA2----SMA Logic<-------------+  |A |B  |L                   |2  |2 |2 |2 |
                                  |  |   |K                   |3  |2 |1 |0 |
                                  |  |   v                    v   |  v  |  v
SFP-------------------------------+--+-<-+                   PHC(                  ) 
SFP-------------------------------+--+                       |
SFP-------------------------------+--+-----------------------+
SFP-------------------------------+--+

The timing information from the DPLL (that can arrive from the GNSS module, SMA connectors,
E810 SDPs, or from the recovered SyncE signals) are routed to the E810 and can be used to
synchronize the PHC (PTP hardware clocks). All E810 ports share only one physical PHC, and this is
particularly useful in creating a Boundary Clock (BC) functionality like what is defined in ITU-T
G.8273.2 (but without SyncE).

No, phase is not absolute time. There is phase synchronization, frequency synchronization and time synchronization. 
So two clocks can be running at the same speed (frequency) but it they don "tick" at the same time their phases are not aligned. 
That is what phase synchronization provides.

The output from the 1PPS from the GNSS receiver in the E810 can be configured to provide phase synchronization to the DPLL1.  
The T-GM use case leveraging time reference signal from GNSS satellite depends on that.
相位同步的前提是频率同步。1PPS即可以同步频率也可以同步相位.(实际上频率同步是通过计算相位变化速率实现的. 
DPLL可以接受任何引号，比如1pps或者RCLK的信号，去达到相位同步?
RCLK从网络上获取频率，并且把相应的频率信号发送给DPLL，当然所有这种信号都包含频率信息和相位信息，所以DPLL可以使用这个信号去lock自己?

https://www.nist.gov/pml/time-and-frequency-division/popular-links/time-frequency-z/time-and-frequency-z-p#:~:text=The%20time%20interval%20for%201,time%20shift%20of%20555%20picoseconds)

DPLL0 is used for generating high stability clock signal and DPLL1 used for driving the output signals.
DPLL0用于生成内部使用的时钟信号(frequency only)，DPLL1用于生成1PPS信号.
ECC (DPLL0) driving the internal clocks and PPS (DPLL1) driving all 1PPS signals.
EEC - DPLL0 = Ethernet equipment clock source from DPLL0 for frequency adjustments., glitchless.
PPS - DPLL1 = 1 PPS generation from DPLL1 for phase adjustments. Glitches allowed. Slower locking.
为什么DPLL1的1PPS输出不能用来同步frequency？DPLL就是同步到GNSS的1PPS输出啊？是因为网卡更容易通过时钟信号同步frequency？

Set periodic output on SDP20 (to synchronize the DPLL1 to the E810 PHC synced by ptp4l): # echo 1 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
Or, if users want to set SDP22 (to synchronize the DPLL0 to E810 PHC synced by ptp4l): # echo 2 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
Set the periodic output on SDP20 (to synchronize the DPLL1 to the E810 PHC synced by ptp4l): # echo 1 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
Or, if users want to set SDP22 (to synchronize the DPLL0 and DPLL1 to the E810 PHC synced by ptp4l):
# echo 2 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period

The Linux kernel provides the standard interface for controlling external synchronization pins
To check if the kernel has the required PTP and pin interface, run the following command:
# cat /proc/kallsyms | grep ptp_pin

In the following example, the driver exposes the ptp7 device:
#ls -R /sys/class/ptp/*/pins/
/sys/class/ptp/ptp7/pins/:
GNSS SMA1 SMA2 U.FL1 U.FL2

In the following example, the ens260f0 net interface exposes pins through the ptp7 interface:
#ls -R /sys/class/net/*/device/ptp/*/pins
/sys/class/net/ens260f0/device/ptp/ptp7/pins:
GNSS SMA1 SMA2 U.FL1 U.FL2

Users can also run ethtool -T <interface name> to show the PTP clock number.
# ethtool -T <interface name>
PTP Hardware Clock: 7
```

### 设置DPLL pin
```
The E810-XXVDA4T has four connectors for external 1PPS signals: SMA1, SMA2, U.FL1, and U.FL2
• SMA connectors are bidirectional and U.FL are unidirectional.
• U.FL1 is 1PPS output and U.FL2 is 1PPS input.
• SMA1 and U.FL1 connectors share channel one.
• SMA2 and U.FL2 connectors share channel two.

echo <function> <channel> > /sys/class/net/$ETH/device/ptp/*/pins/SMA1(SMA2,U.FL1,U.FL2)
function: 
  0 = Disabled
  1 = Rx
  2 = Tx
channel: 
  1 = SMA1 or U.FL1
  2 = SMA2 or U.FL2

channel 1:
1. SMA1 as 1PPS input:
# echo 1 1 > /sys/class/net/$ETH/device/ptp/ptp*/pins/SMA1
2. SMA1 as 1PPS output:
# echo 2 1 > /sys/class/net/$ETH/device/ptp/ptp*/pins/SMA1
channel 2:
1. SMA2 as 1PPS input:
# echo 1 2 > /sys/class/net/$ETH/device/ptp/ptp*/pins/SMA2
2. SMA2 as 1PPS output:
# echo 2 2 > /sys/class/net/$ETH/device/ptp/ptp*/pins/SMA2
```
### Recovered Clocks (G.8261 SyncE Support)  
```
Recovered clocks can be configured using a special sysfs interface that is exposed by every port
instance. Writing to a sysfs under a given port automatically enables a recovered clock from a given
port that is valid for a current link speed. A link speed change requires repeating the steps to enable the
recovered clock.

If a port recovered clock is enabled and no higher-priority clock is enabled at the same time, the DPLL
starts tuning its frequency to the recovered clock reference frequency enabling G.8261 functionality.
There are two recovered clock outputs from the C827 PHY. Only one pin can be assigned to one of the
ports. Re-enabling the same pin on a different port automatically disables it for the previously-assigned
port.

1. To enable a recovered clock for a given Ethernet device run the following:
# echo <enable> <pin(clock id)> > /sys/class/net/$ETH/device/phy/synce
where:
	ena: 0 = Disable the given recovered clock pin.
	     1 = Enable the given recovered clock pin.
	pin: 0 = Enable C827_0-RCLKA (higher priority pin).
	     1 = Enable C827_0-RCLKB (lower priority pin).

For example, to enable the higher-priority recovered clock from Port 0 and a lower-priority recovered clock from Port 1, run the following:
# export ETH0=enp1s0f0
# export ETH1=enp1s0f1
# echo 1 0 > /sys/class/net/$ETH0/device/phy/synce
# dmesg
[27575.495705] ice 0000:03:00.0: Enabled recovered clock: pin C827_0-RCLKA
# echo 1 1 > /sys/class/net/$ETH1/device/phy/synce
# dmesg
[27575.495705] ice 0000:03:00.0: Enabled recovered clock: pin C827_0-RCLKB

2.Disable recovered clocks:
# echo 0 0 > /sys/class/net/$ETH0/device/phy/synce
# dmesg
[27730.341153] ice 0000:03:00.0: Disabled recovered clock: pin C827_0-RCLKA
# echo 0 1 > /sys/class/net/$ETH1/device/phy/synce
# dmesg
[27730.341153] ice 0000:03:00.0: Disabled recovered clock: pin C827_0-RCLKB

Check recovered clock status:
You can add the current status of the recovered clock to the dmesg:
#echo dump rclk_status > /sys/kernel/debug/ice/0000:03:00.0/command
# dmesg
[311274.298749] ice 0000:03:00.0: State for port 0, C827_0-RCLKA: Disabled
[311274.300060] ice 0000:03:00.0: State for port 0, C827_0-RCLKB: Disabled
```

### External Timestamp Signals
```
The E810-XXVDA4T can use external 1PPS signals filtered out by the DPLL as its own time reference.
When the DPLL is synchronized to the GNSS module or an external 1PPS source, the ts2phc tool can be
used to synchronize the time to the 1PPS signal.

# export ETH=enp1s0f0
# export TS2PHC_CONFIG=/home/<user>/linuxptp-3.1/configs/ts2phc-generic.cfg
# ts2phc -f $TS2PHC_CONFIG -s generic -m -c $ETH
# cat $TS2PHC_CONFIG
[global]
use_syslog 0
verbose 1
logging_level 7
ts2phc.pulsewidth 100000000
#For GNSS module
#ts2phc.nmea_serialport /dev/ttyGNSS_BBDD_0 #BB bus number DD device number /dev/
ttyGNSS_1800_0
#leapfile /../<path to .list leap second file>
[<network interface>]
ts2phc.extts_polarity
rising
```

### Periodic Outputs From DPLL (SMA and U.FL Pins)
```
The E810-XXVDA4T supports two periodic output channels (SMA1 or U.FL1 and SMA2). Channels can be
enabled independently and output 1PPS generated by the embedded DPLL. 1PPS outputs are
synchronized to the reference input driving the DPLL1. Users can read the current reference signal
driving the 1PPS subsystem by running the following command:

# dmesg | grep "<DPLL1> state changed" | grep locked | tail -1
[ 342.850270] ice 0000:01:00.0: DPLL1 state changed to: locked, pin GNSS-1PPS

The following configurations of 1PPS outputs are supported:
1. SMA1 as 1PPS output:
# echo 2 1 > /sys/class/net/$ETH/device/ptp/ptp*/pins/SMA1
2. U.FL1 as 1PPS output:
# echo 2 1 > /sys/class/net/$ETH/device/ptp/ptp*/pins/U.FL1
3. SMA2 as 1PPS output:
# echo 2 2 > /sys/class/net/$ETH/device/ptp/ptp*/pins/SMA2
```

###  Reading Status of the DPLL
```
# cat /sys/kernel/debug/ice/$PCI_SLOT/cgu
Found ZL80032 CGU
DPLL Config ver: 1.3.0.1
CGU Input status:
 | | priority |
 input (idx) | state | EEC (0) | PPS (1) |
 ---------------------------------------------------
 CVL-SDP22 (0) | invalid | 8 | 8 |
 CVL-SDP20 (1) | invalid | 15 | 3 |
 C827_0-RCLKA (2) | invalid | 4 | 4 |
 C827_0-RCLKB (3) | invalid | 5 | 5 |
 SMA1 (4) | invalid | 1 | 1 |
 SMA2/U.FL2 (5) | invalid | 2 | 2 |
 GNSS-1PPS (6) | valid | 0 | 0 |
EEC DPLL:
Current reference: GNSS-1PPS
Status: locked_ho_ack
PPS DPLL:
Current reference: GNSS-1PPS
Status: locked_ho_ack
Phase offset: -217

The first section of the log shows the status of CGU inputs (references) including its index number.
Active references currently selected are listed in Section 4.2. EEC Ethernet equipment clock (DPLL0)
skips the 1PPS signal received on the CVL-SDP20 pin.
The second section lists all internal DPLL units. ECC (DPLL0) driving the internal clocks and PPS (DPLL1)
driving all 1PPS signals.
```

### DPLL Monitoring
```
In the default configuration, the E810-XXVDA4T driver enables monitoring of the DPLL events and reports state changes 
in the default system log (dmesg) with the WARN level independently for each of the DPLL units.
DPLLs start in a holdover mode and enter an unlocked and locked state when a valid reference input is enabled. 
If the current input becomes invalid, DPLLs change state to holdover. When the reference reappears (or a different valid input is present), 
the DPLL state changes to the unlocked state and locks to a new signal.

Enabling DPLL monitoring:
ethtool --set-priv-flags $ETH dpll_monitor on
Disabling DPLL monitoring:
ethtool --set-priv-flags $ETH dpll_monitor off
```

### pin_cfg User Readable Format
```
To check the DPLL pin configuration:
# cat /sys/class/net/ens4f0/device/pin_cfg
in
| pin| enabled| freq| phase_delay| esync| DPLL0 prio| DPLL1 prio|
| 0| 1| 1| 0| 0| 8| 8|
| 1| 1| 1| 0| 0| 15| 3|
| 2| 1| 1953125| 0| 0| 4| 4|
| 3| 1| 1953125| 0| 0| 5| 5|
| 4| 1| 1| 7000| 0| 1| 1|
| 5| 1| 1| 7000| 0| 2| 2|
| 6| 1| 1| 0| 0| 0| 0|
out
| pin| enabled| dpll| freq| esync|
| 0| 1| 1| 1| 0|
| 1| 1| 1| 1| 0|
| 2| 1| 0| 156250000| 0|
| 3| 1| 0| 156250000| 0|
| 4| 1| 1| 1| 0|
| 5| 1| 1| 1| 0|
In the “in” table. the pin numbers are referred from the DPLL Priority See Section 4.2, “DPLL Priority”.
In the “out” table pin 0 is SMA1 pin 1 is SMA2, all the other values do not modify.

Changing the DPLL priority list:
# echo "prio <prio value> dpll <dpll index> pin <pin index>" > \ /sys/class/net/<dev>/device/pin_cfg
where:
prio value = Desired priority of configured pin [0-14]
dpll index = Index of DPLL being configured [0:EEC (DPLL0), 1:PPS (DPLL1)]
pin index = Index of pin being configured [0-9]

Example:
Set priority 1 for pin 3 on DPLL 0:
# export ETH=enp1s0f0
# echo "prio 1 dpll 0 pin 3" > /sys/class/net/$ETH/device/pin_cfg

Changing input/output pin configuration:
# echo "<direction> pin <pin index> <config>" > /sys/class/net/<dev>/device/pin_cfg
where:
direction = pin direction being configured [“in”: input pin, “out”: output pin]
pin index = index of pin being configured [for in 0-6 (see DPLL priority section); for out 0:SMA1 1: SMA2]
config = list of configuration parameters and values:
[ "freq <freq value in Hz>",
 "phase_delay <phase delay value in ns>" // NOT used for out,
 "esync <0:disabled, 1:enabled>"
 "enable <0:disabled, 1:enabled>" ]

Note: The esync setting has meaning only with the 10 MHz frequency, you need to have esync to have the same setting in both ends of the SMA.
Example:
# export ETH=enp1s0f0
Set freq to 10 MHz on input pin 4: DPLL will lock only if 10 MHz signals arrive on SMA1 and it has been enabled for input.
# echo "in pin 4 freq 10000000" > /sys/class/net/$ETH/device/pin_cfg

```

### cgu_ref_pin/cgu_state Machine Readable Interface
```
To find out which pin the DPLL0 (EEC DPLL) is locked on, check the cgu_ref_pin:
# cat /sys/class/net/<dev>/device/cgu_ref_pin
To check the state of the DPLL0 (EEC DPLL) you can check the cgu_state:
# cat /sys/class/net/<dev>/device/cgu_state
DPLL_UNKNOWN = -1,
DPLL_INVALID = 0,
DPLL_FREERUN = 1,
DPLL_LOCKED = 2,
DPLL_LOCKED_HO_ACQ = 3,
DPLL_HOLDOVER = 4
The cgu_state interface used by synce4l as well.
```

### 1PPS Signals from E810 Device to DPLL
```
The E810-XXVDA4T implements two 1PPS signals coming out of the MAC (E810 device) to the DPLL.
They serve as the phase reference (CVL-SDP20) and as both phase and frequency reference (CVLSDP22) signals.
To enable a periodic output, write five integers into the file: channel index, start time seconds, start
time nanoseconds, period seconds, and period nanoseconds. To disable a periodic output, set all the
seconds and nanoseconds values to zero.

1. To enable the phase reference pin (CVL-SDP20):
# echo 1 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period

2. To enable the phase and frequency reference pin (CVL-SDP22):
# echo 2 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period

3. To disable the phase reference pin (CVL-SDP20):
# echo 1 0 0 0 0 > /sys/class/net/$ETH/device/ptp/ptp*/period

4. To disable the phase and frequency reference pin (CVL-SDP22):
# echo 2 0 0 0 0 > /sys/class/net/$ETH/device/ptp/ptp*/period

Set periodic output on SDP20 (to synchronize the DPLL1 to the E810 PHC synced by ptp4l): # echo 1 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
Or, if users want to set SDP22 (to synchronize the DPLL0 to E810 PHC synced by ptp4l): # echo 2 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
Set the periodic output on SDP20 (to synchronize the DPLL1 to the E810 PHC synced by ptp4l): # echo 1 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
Or, if users want to set SDP22 (to synchronize the DPLL0 and DPLL1 to the E810 PHC synced by ptp4l):
# echo 2 0 0 1 0 > /sys/class/net/$ETH/device/ptp/ptp*/period
```

### 1PPS Signals from the DPLL to E810 Device
```
The DPLL automatically delivers 2x 1PPS signals to the E810 device on pin 21 and 23. These signals can
be used to synchronize the E810 to the DPLL phase with the ts2phc program. The E810 will capture the
timestamp when the 1PPS signal arrives.

To ensure that the timestamps from the 1PPS signals are only used when the DPLL is locked to a signal,
you can enable the 1 PPS filtering option. By default, filtering is disabled, and all the timestamps are
passed along by the E810 ice driver.

The DPLL 1PPS filtering can be enabled (on) or disabled (off) by using the ethtool command in the Linux
kernel.

# export ETH=ens801f0
#ethtool --show-priv-flags $ETH
Private flags for ens801f0:
link-down-on-close : off
fw-lldp-agent : off
channel-inline-flow-director : off
channel-inline-fd-mark : off
channel-pkt-inspect-optimize : on
channel-pkt-clean-bp-stop : off
channel-pkt-clean-bp-stop-cfg: off
vf-true-promisc-support : off
mdd-auto-reset-vf : off
vf-vlan-pruning : off
legacy-rx : off
dpll_monitor : on
extts_filter : off
Enabling DPLL's 1 filtering:
ethtool --set-priv-flags $ETH extts_filter on
Disabling DPLL's 1 filtering:
ethtool --set-priv-flags $ETH extts_filter off
```

## From Intel DPLL TestPlan
```
dpll pin 有三种，
1.input：
用来设置dpll的输入，可以把CVL,SMA,U.FL2,GPS或者RCLK作为dpll输入
通过CVL-SDP可以把ptp作为input。
pin-parent-device的id是dpll序号

2.output：
输出引脚，用来设置dpll的输出.
包括REF-SMA1,REF-SMA2,U.FL1,PHY-CLK,MAC-CLK,CVL-SDP

3.synce-eth-port：
input pin中的RCLKA和RCLKB是multiplexer类型的pin。
synce-eth-port可以注册到这种multiplexer类型的pin，注册之后，他的parent就是multiplexer类型的pin。
一个synce-eth-port可以注册到多个mux pins。
多个synce-eth-port可以注册到同一个mux pins，但是同一时刻只有一个synce-eth-port pin可以作为mux pin的input。
可以通过 设置state来指定哪个pin作为input生效。
比如 Enable RCLKA on pin-id 13 (RCLKA is ("pin-parent-pin":{"pin-id":2))
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":13, "pinparent-pin":{"pin-id":2, "pin-state":1}}'

There are 2 recovery clocks on each device RCLKA and RCLKB

RCLKA can only be connected to one port at a time and RCLKB can only be connected to one port at
a time. Both can be assigned to the same port at the same time. Which one is connected is based on
priority of the pin

Enable RCLKA on pin-id 13 (RCLKA is ("pin-parent-pin":{"pin-id":2))
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":13, "pinparent-pin":{"pin-id":2, "pin-state":1}}'
上面的命令有问题，下面可以执行
# ./cli.py --spec /root/dpll.yaml --schema /root/genetlink.yaml --do pin-set --json '{"id":13, "parent-pin":{"parent-id":2, "state":1}}'
{'capabilities': 4,
  'clock-id': 5799633565436792966,
  'id': 13,
  'module-name': 'ice',
  'parent-pin': [{'parent-id': 2, 'state': 'connected'},
                 {'parent-id': 3, 'state': 'disconnected'}],
  'type': 'synce-eth-port'},

Use the pin-get command to see the status:
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --dump pin-get

Verify RCLKA ‘input’ (pin id 2) is connected using the pin-get command:
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --dump pin-get

Verify dpll status changes from unlocked/holdover to locked/locked-ho-acq
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --dump device-get

Enable RCLKB on pin-id 15 (RCLKB is ("pin-parent-pin":{"pin-id":3))
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":15, "pinparent-pin":{"pin-id":3, "pin-state":1}}'

Notice RCLKB input pin id 3 will not be connected until the you change its priority on either
dpll 0 or 1 to be higher than RCLKA. Current priority is 8 for RCLKA for dpll 0 and 1 and
current priority for RCLKB for dpll 0 and 1 is 9. So change RCLKB prio to 7.
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":3, "pin-parentdevice":{"id":1, "pin-prio":7}}'
上面命令不能执行，下面可以执行
# ./cli.py --spec /root/dpll.yaml --schema /root/genetlink.yaml --do pin-set --json '{"id":6, "parent-device":{"parent-id":1, "prio":5}}'

Should see RCLKB dpll 1 (pin-parent-device – id 1) show connected and RCLKA dpll 1 (pinparent-device – id 1) be selectable now. 
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-get --json '{"pin-id":3}'
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-get --json '{"pin-id":2}


Disable recovery on both port pins:
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":13,
"pin-parent-pin":{"pin-id":2, "pin-state":3}}'
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":15,
"pin-parent-pin":{"pin-id":3, "pin-state":3}}'

Verify recovery is disabled on both port pins:
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --dump pin-get
# 

Verify dpll is in holdover state:
./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --dump device-get

Enable recovery on both port pins again:
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":13,
"pin-parent-pin":{"pin-id":2, "pin-state":1}}'
# ./cli.py --spec /usr/src/kernels/linux-dpll-net-next-dpllv11/Documentation/netlink/specs/dpll.yaml --do pin-set --json '{"pin-id":15,
"pin-parent-pin":{"pin-id":3, "pin-state":1}}'

synce configuration:
server node external_input=1 internal_input=0
follower node external_input=0 internal_input=1
在synce配置文件里面添加dpll_state_file
在synce配置文件里面添加port，并为每个port设置recover_clock_file
启动synce4l on each side:
synce4l -f conifgs/synce.cfg -m

Go to /sys/class/net/<device name>/device/phy folder and execute the following command to enable
RCLKA as follows,
Note - do: echo <ENABLE (1) or DISABLE (0)> <WHICH RCLK, either 0 or 1>
# echo 1 0 > synce

Again DPLL status and RCLKA pin should be valid now,
# cat /sys/kernel/debug/ice/0000\:18\:00.0/dpll

turn off SW pin of dpll , it'll trigger DNU for quality signal and ext QL = 255 or 0xff
# echo 0 0 > synce
```

## ts2phc
```
-s generic 
  Use the key word "generic" for an external 1-PPS without ToD information.
  1PPS from DPLL1
-s nmea
  Use the key word "nmea" for an external 1-PPS from a GPS providing ToD information via the RMC NMEA sentence.
  就是GNSS console ToD

如果用generic，就只需SDP21或者23
如果用nmea，那么还需要console ToD。

GNSS recvr syncs itself from the GNSS constellation(s) → 
GNSS syncs (DPLL0) and DPLL1 → 
syncs E810’s PHC (with ts2phc, using ToD NMEAs from the vSerial of GNSS + 1PPS out from DPLL1 / 1PPS in E810 SDP21|23) → 
phc2sys sync node clock fom PHC

The phc2sys tool should not be run at the same time as ts2phc using the generic source of ToD (-s generic). 
In the default configuration, ts2phc uses hardware-generated timestamps along with the system timer to create correction values. 
Running the tools in parallel can create a feedback that breaks time synchronization. 
The leapfile option is available but not necessary for the program to run. 
Also, the default .leap file is not compatible with ts2phc.

```

## synce4l
```
https://github.com/intel/synce4l

The lack of messages is considered to be a failure condition. 
The protocol behaviour is such that the quality level is considered QL-FAILED if no SSM messages are received after a five second period.

wait-to-restore:
The wait-to-restore time ensures that a previous failed synchronization source is only again considered as available by the selection process if it is fault-free for a certain time.
就是必须恢复正常多长时间之后才认为他真正有效。
recover_time和wait-to-restore是同样的意思。
停止master synce4l之后，slave side会因为5秒内未收到SSM而进入QL-FAILED状态。
再启动master synce4l之后，slave在收到SSM消息之后，要等待recover_time才能切换为lock状态

network_option:
Network option according to T-REC-G.8264. All devices in SyncE domain should have the same option configured.

tx_heartbeat_msec:
ESMC发送间隔

rx_heartbeat_msec:
ESMC poll间隔

use_syslog 1
message_tag                [synce4l]
vim /etc/rsyslog.conf 
:syslogtag, contains, "synce4l"                         /var/log/synce

input_QL                   0x1   // SSM code https://github.com/intel/synce4l
input_ext_QL               0x20  // Extended SSM code https://github.com/intel/synce4l

```

## DPLL 核心架构和部件
```
DPLL 核心思想
DPLL 的核心目标与模拟锁相环 (APLL) 相同：产生一个输出信号，使其频率和相位与一个输入的参考信号保持精确的同步。
但 DPLL 的关键区别在于，它所有的处理过程——从相位比较到环路滤波再到振荡器控制——都是在数字域使用数字信号处理 (DSP) 技术完成的。

主要架构 (Conceptual Block Diagram)
一个典型的DPLL系统由以下四个核心部件构成一个闭环反馈系统：

           +-----------------+      +--------------------+      +-----------------+
参考输入 -->|  鉴相器 (PD)   |----->|  数字环路滤波器     |----->|  数控振荡器      |--> 输出时钟
 (Ref In)  | Phase Detector  |      | (Digital Loop Filter)|      | (DCO / NCO)     |  (Output)
           +-------^---------+      +--------------------+      +-------+---------+
                   |                                                    |
                   |             +-------------------+                  |
                   +-------------|  反馈分频器 (N)    |<-----------------+
                                 | Feedback Divider  |
                                 +-------------------+
下面我们详细介绍每一个主要部件的作用。

主要部件详解
1. 鉴相器 (Phase Detector, PD) / 鉴频鉴相器 (PFD)
角色: DPLL的“眼睛”。

功能: 这是整个环路的起点。它的任务是精确地比较输入的参考时钟信号和来自本地振荡器的反馈时钟信号之间的相位差异（以及频率差异）。

工作原理: 在数字系统中，鉴相器不再是简单的模拟混频器。它通常是一个时间-数字转换器 (Time-to-Digital Converter, TDC)。TDC 会测量参考时钟的上升沿和反馈时钟的上升沿之间极其微小的时间间隔。这个时间差随后被转换成一个数字值（相位误差信号），并传递给下一个部件。

2. 数字环路滤波器 (Digital Loop Filter, DLF)
角色: DPLL的“大脑”。

功能: 这是DPLL智能化的核心。它接收来自鉴相器的一系列离散的、数字化的相位误差信号，并执行复杂的数字滤波算法。其主要目标是：

平滑误差: 滤除参考时钟本身带来的噪声和抖动 (Jitter)，只关注长期、稳定的相位趋势。

决定响应特性: 控制DPLL的“带宽”。窄带宽可以更好地抑制噪声，但锁定速度慢；宽带宽锁定速度快，但噪声抑制能力差。

确保环路稳定: 通过算法（如PI/PID控制器）计算出精确的校正量，避免对振荡器的调整过冲或不足，防止环路发生振荡。

实现高级功能: 在这里实现保持 (Holdover) 模式的逻辑。当参考输入丢失时，环路滤波器会“记住”最后一次的校正值，以维持振荡器的稳定输出。

工作原理: 它是一个纯粹的数字信号处理 (DSP) 模块，在硬件逻辑或软件中实现。它接收相位误差序列，并输出一个数字控制字 (Digital Control Word)。

3. 数控振荡器 (Numerically/Digitally-Controlled Oscillator, NCO/DCO)
角色: DPLL的“心脏”。

功能: 这是实际产生输出时钟信号的本地时钟源。它的频率可以被来自环路滤波器的数字控制字进行精确地、离散地调整。

工作原理:

DCO (数字控制振荡器): 通常是一个压控晶体振荡器 (VCXO)，但其控制电压由一个高精度的数模转换器 (DAC) 产生。环路滤波器输出的数字控制字被DAC转换成模拟电压来“拉动”晶振的频率。

NCO (数值控制振荡器): 这是一个完全的数字电路。它内部有一个“相位累加器”，在每个时钟周期累加一个固定的频率控制字。累加器的值代表了波形的相位，然后通过查找表或其他逻辑生成最终的方波或正弦波输出。NCO的频率分辨率极高，调整非常线性。

4. 反馈分频器 (Feedback Divider, N)
角色: 连接输出与输入的桥梁。

功能: 它将NCO/DCO产生的高频输出时钟进行整数分频，生成一个频率与参考时钟相近的反馈信号，送回到鉴相器进行比较。

工作原理: 这是一个简单的数字计数器。通过改变分频比 N，DPLL可以实现频率合成——即输出一个频率为参考时钟 N 倍的时钟信号。

整体工作流程
比较: 鉴相器比较参考时钟和经过分频的本地时钟，产生一个代表相位误差的数字。

思考: 数字环路滤波器处理这个误差，并结合历史数据，计算出一个最优的数字校正指令。

调整: 数控振荡器接收这个指令，微调其输出频率。

反馈: 调整后的输出经过分频器，再次送回鉴相器。

循环: 这个过程以极高的速度不断重复，每一次循环都让相位误差变得更小，直到最终本地时钟的相位和频率与参考时钟完全“锁定”。

```

## dpll有自己的晶体振荡器吗？也就是说他有自己的时钟源吗？
```
是的，可以这么理解。更精确地说，一个 DPLL（数字锁相环）的核心功能就是去控制一个本地的振荡器，这个振荡器就是它内部的时钟源。

为了更好地理解，我们来分解一下：

1. DPLL 必须包含一个本地振荡器
一个完整的 DPLL 系统必然包含一个本地的、能够产生时钟信号的组件。这个组件通常是：

DCO (Digitally Controlled Oscillator): 数字控制振荡器。

NCO (Numerically Controlled Oscillator): 数值控制振荡器。

这个 DCO/NCO 就是 DPLL 的“心脏”，是实际产生输出时钟的源头。在很多实际的 DPLL 芯片设计中，这个 DCO/NCO 需要外接一个**晶体（Crystal）**作为其物理振荡的基础。所以，当您看到一个 DPLL 芯片的电路图时，几乎总会看到它旁边连接着一个晶体振荡器 (XO)。

所以，您的第一个问题的直接答案是：是的，一个 DPLL 系统拥有自己的晶体振荡器作为其本地时钟源。

2. DPLL 的目的不是“独立”，而是“同步”
这是最关键的一点。虽然 DPLL 有自己的时钟源，但它的主要目的不是让这个时钟源自由运行（Freerun），而是用一个外部的、更精确的**参考时钟（Reference Clock）**来“锁定”和“校准”它。

DPLL 的工作模式清晰地说明了这一点：

锁定模式 (Locked Mode): 这是正常工作状态。DPLL 持续地将自己的本地振荡器与输入的参考时钟进行比较，并不断微调本地振荡器，使其频率和相位与参考时钟保持严格同步。此时，输出时钟的长期精度等同于参考时钟的精度。

保持模式 (Holdover Mode): 当外部的参考时钟丢失时，DPLL 会切换到此模式。它会利用丢失前最后一次“锁定”时的校准数据，尽力维持本地振荡器的频率。此时，它就是在依靠“自己”的时钟源运行，但会随着时间和温度的变化慢慢产生漂移。

自由运行模式 (Freerun Mode): 如果从未有过有效的参考时钟输入，DPLL 就会工作在此模式。本地振荡器会按照其未经校准的、自然的频率运行。此时它的精度通常不高，相当于一个普通的晶体振荡器。

总结与比喻
您可以把 DPLL 想象成一个高水平的乐队乐手：

本地振荡器 (DCO/XO): 是乐手自己的节拍感和演奏能力。他自己当然能打出节拍。

外部参考时钟: 是指挥家的指挥棒。

DPLL 的数字逻辑: 是乐手的大脑和眼睛。

当指挥家在场时（锁定模式），乐手会全神贯注地跟随指挥棒，确保自己的演奏与整个乐队完美同步。当指挥家突然离开（保持模式），乐手会凭记忆尽力保持刚才的节拍，但时间一长可能会有偏差。如果一开始就没有指挥家（自由运行模式），乐手就只能按自己的感觉来演奏了。

因此，结论是：

是的，DPLL 有自己的时钟源（通常是晶体振荡器），但它的价值在于能够利用其复杂的数字逻辑，将这个时钟源精确地同步到一个更高质量的外部参考源上。

```

## DPLL只接收1PPS输入，或者只接收高频率时钟输入，对它来说有什么区别？这两种情况下DPLL都可以正常工作吗？
```
这是一个非常核心的问题，它触及了DPLL在不同应用场景下的工作原理。

简单来说，答案是：是的，DPLL在这两种情况下都可以正常工作，但它所实现的“同步”目标完全不同，性能表现也有天壤之别。 理想情况下，一个高精度的DPLL会同时使用这两种输入。

下面我们来详细分解这两种情况的区别。

情况一：只接收 1PPS 输入 (实现“相位同步”)
当DPLL只接收一个1PPS（每秒脉冲）信号时，它主要在进行相位同步或时间同步。

能否工作？
能。这是DPLL的一种常见工作模式。

工作原理：

DPLL使用其内部的本地振荡器（例如一个TCXO或OCXO）来生成一个高频率的输出时钟。

每当1PPS参考脉冲到达时，DPLL的相位检测器会比较本地振荡器产生的“秒脉冲”与外部输入的1PPS脉冲之间的时间差（相位误差）。

根据这个误差，DPLL的环路滤波器会计算出一个修正值，去微调本地振荡器的频率，使其产生的下一个秒脉冲能更准时地对齐外部1PPS。

主要区别和局限性：

“准星”稀疏： DPLL每秒只有一次校准其相位的机会。在两次1PPS脉冲之间的漫长间隔（近1秒）里，输出时钟的频率完全依赖于其本地振荡器的稳定性。

性能依赖本地振荡器： 如果本地振荡器不稳定（温漂、老化），那么在1秒的间隔内，输出时钟的频率可能会发生显著漂移。这会导致输出信号产生较大的抖动（Jitter）和漂移（Wander）。

收敛速度慢： 因为每秒只能调整一次，所以DPLL需要更长的时间才能完全锁定到1PPS的相位上。

可以把1PPS想象成一个狙击手每秒只能看一次准星，虽然长期来看能保证目标方向正确，但中间过程的稳定性完全取决于他自己手臂的稳定程度。

情况二：只接收高频率时钟输入 (实现“频率同步”)
当DPLL只接收一个高频时钟（如10MHz, 25MHz）时，它主要在进行频率同步。

能否工作？
能。这是锁相环最经典的工作模式。

工作原理：

DPLL的相位检测器会非常频繁地（每秒数百万次）比较本地振荡器的输出与外部高频参考时钟的相位。

任何微小的频率偏差都会被迅速检测到并校正。这使得本地振荡器的频率能够被非常紧密地“锁死”在参考时钟的频率上。

主要区别和局限性：

频率稳定，相位模糊： DPLL可以完美复制参考时钟的频率稳定性，输出一个非常平滑、低抖动的时钟信号。

缺少绝对时间信息： 它只知道时钟的“节拍”有多快，但不知道哪个节拍是“一秒钟”的开始。输出时钟的相位与UTC（协调世界时）的秒边界可能是随机对齐的。

典型应用： 这种模式非常适用于**同步以太网（SyncE）**等场景，这些应用只需要一个稳定的频率来同步数据传输，而不在乎绝对时间。

这就像一个乐手能完美跟上节拍器的速度，但并不知道哪一拍是乐章的开始。

理想情况：同时使用 1PPS 和高频时钟
现代高性能DPLL（尤其是在电信、5G和PTP应用中）被设计为同时接收这两种输入，以实现最完美的时间和相位同步。

工作原理：

DPLL主要使用高频时钟输入来 disciplining（驯服）本地振荡器的频率，确保输出信号平滑、稳定、抖动极低。

同时，它使用1PPS输入作为绝对的时间参考，来缓慢地“校准”输出时钟的相位。

最终，DPLL能够生成一个既有极高频率稳定性，又能精确对齐UTC秒边界的高质量时钟信号。

总结对比
输入类型    主要同步目标	优点	                        缺点/局限性
仅 1PPS	   相位/时间同步	长期相位准确，能对齐UTC秒	短期频率稳定性差，依赖本地振荡器，抖动和漂移较大
仅高频时钟	频率同步	短期频率稳定性极好，抖动低	缺乏绝对时间信息，相位与UTC无关
1PPS + 高频时钟	时间、相位和频率同步	兼具两者的优点，输出高质量、低抖动、且相位精确的时钟	对DPLL芯片和系统设计要求更高

Export to Sheets
结论： 两种情况DPLL都可以工作，但实现的目标和性能表现截然不同。只输入1PPS是为了对准“时间”，只输入高频时钟是为了对准“频率”。对于需要高精度时间的应用（如PTP），同时使用两者是业界标准做法。

```

## EEC and PPS

```
# /mnt/dpll/ynl/pyynl/cli.py --spec=/mnt/dpll/dpll.yaml --no-schema --dump device-get

[{'clock-id': 5799633565437511660,

  'id': 0,

  'lock-status': 'locked-ho-acq',

  'mode': 'automatic',

  'mode-supported': ['automatic'],

  'module-name': 'ice',

  'type': 'eec'},

 {'clock-id': 5799633565437511660,

  'id': 1,

  'lock-status': 'locked-ho-acq',

  'mode': 'automatic',

  'mode-supported': ['automatic'],

  'module-name': 'ice',

  'type': 'pps'}]

这个机器上的dpll信息怎么解释？eec和pps是两个不同的dpll吗？




核心问题解答
首先，回答您的核心问题：

EEC 和 PPS 不是两个不同的物理 DPLL 芯片，它们代表的是同一个高级 DPLL 硬件模块所扮演的两种不同角色或管理的两种不同类型的时钟同步功能。

将它们视为同一个复杂DPLL引擎的两个逻辑接口或功能单元会更容易理解。

信息解释
让我们逐条分析输出的含义，特别是那些关键的字段：

1. clock-id: 5799633565437511660
含义： 这是物理DPLL硬件模块的唯一标识符。

关键点： 您会发现两个条目中的 clock-id 是完全相同的。这是最有力的证据，证明它们指向的是同一个物理设备。如果它们是两个独立的DPLL芯片，这个ID将会不同。

2. id: 0 和 id: 1
含义： 这是在同一个 clock-id 下的逻辑单元ID。操作系统（Linux的DPLL子系统）使用这个ID来区分同一个物理硬件所提供的不同功能。

3. type: 'eec' 和 type: 'pps' （最重要的区别）
这揭示了该DPLL的两种核心功能：

type: 'eec' (Ethernet Equipment Clock - 以太网设备时钟)

功能： 这部分功能专门用于频率同步 (Frequency Synchronization)。

技术背景： 它与 同步以太网 (Synchronous Ethernet, SyncE) 技术紧密相关。SyncE通过从以太网物理层（PHY）恢复时钟信号来获得一个非常稳定的频率参考。这个功能的目标是确保数据传输的比特率是稳定和统一的，但它不关心绝对时间（现在是几点几分几秒）。

type: 'pps' (Pulse Per Second - 每秒脉冲)

功能： 这部分功能专门用于相位和时间同步 (Phase/Time Synchronization)。

技术背景： 它与 PTP (精确时间协议) 或 GPS 等技术相关。PPS信号是一个非常精确的、每秒一次的脉冲，用于校准时间的起点（秒的边界）。这个功能的目标是确保设备的内部时钟与一个主时钟（如PTP Grandmaster）在时间上精确对齐。

4. lock-status: 'locked-ho-acq'
含义： 这是一个组合状态，表示DPLL当前的锁定状态和能力。

locked: 已锁定。这是最好的状态，意味着DPLL已经成功地同步到了一个有效的外部参考时钟。

ho: 保持 (Holdover)。这表示DPLL具备保持能力。如果外部参考时钟丢失，它可以切换到Holdover模式，依据最后一次的校准数据继续运行一段时间。

acq: 正在采集 (Acquiring)。表示DPLL具备在丢失参考后重新捕获和锁定新参考的能力。

结论： 您的DPLL当前工作状态良好，并且具备高级的保持和重捕获功能。

5. module-name: 'ice'
含义： 这是暴露这个DPLL功能的内核驱动模块的名称。ice 是 Intel® Ethernet Controller E800系列网卡的驱动程序。

结论： 这证实了该DPLL硬件是集成在这张高性能Intel网卡上的。

总结与比喻
您可以将这个DPLL硬件模块想象成一个多功能厨房料理机：

clock-id 是这台料理机的序列号。

EEC功能 (id: 0) 就像是它的“榨汁功能”，专门处理频率同步。

PPS功能 (id: 1) 就像是它的“绞肉功能”，专门处理相位/时间同步。

您看到的是同一台机器（同一个clock-id）对外展示的两种不同能力。操作系统为了方便管理，将这两种能力分别呈现为两个逻辑设备。

总而言之，您的系统拥有一块集成了高级DPLL的Intel网卡，该DPLL能够同时处理用于SyncE的频率同步（eec）和用于PTP的相位同步（pps），并且当前已成功锁定到一个参考源，工作状态正常。
```

## DPLL既可以输出1PPS信号也可以输出时钟信号，这两种信号分别是由哪个部件输出的？
```
Excellent question. Both of these signals ultimately originate from the same core component, but the 1PPS signal is a derived product of the high-frequency clock.

Here is a clear breakdown of which part of the DPLL architecture is responsible for each output:

1. 高频率时钟信号 (High-Frequency Clock Signal)
输出部件: 数控振荡器 (NCO - Numerically Controlled Oscillator / DCO - Digitally Controlled Oscillator)

This is the most direct answer. The high-frequency clock (e.g., 10 MHz, 100 MHz, etc.) is the primary output of the DPLL's core oscillator.

The entire purpose of the DPLL's feedback loop (鉴相器, 数字环路滤波器) is to continuously adjust and stabilize this NCO/DCO, ensuring its output is perfectly locked in frequency and phase to the incoming reference signal. This stable, high-frequency signal is then buffered and provided as a main output of the DPLL chip or system.

2. 1PPS (每秒脉冲) 输出信号
输出部件: 数字分频器 (Digital Frequency Divider) / 可编程计数器 (Programmable Counter)

The 1PPS signal is not generated directly by the main oscillator. Instead, it is created by taking the high-frequency clock from the NCO/DCO and dividing its frequency down.

Here is the process:

源头 (Source): The stable, phase-aligned high-frequency clock is taken from the output of the NCO/DCO.

处理 (Processing): This clock signal is fed into a dedicated digital divider circuit.

生成 (Generation): This divider is programmed to count an exact number of cycles of the high-frequency clock. When the count reaches the number of cycles that corresponds to precisely one second, the circuit outputs a single, sharp pulse. The counter then resets and begins counting again.

举个例子:
如果 NCO 输出一个被完美锁定的 100 MHz 时钟信号 (每秒 100,000,000 个周期)，那么 1PPS 输出模块就会：

接收这个 100 MHz 信号。

开始计数，每收到一个时钟脉冲，计数器加一。

当计数器数到 100,000,000 时，它会产生一个脉冲（1PPS 信号）。

然后计数器归零，并重复此过程。

总结与架构图
所以，这两个信号的产生关系可以这样表示：

                               +-----------------------------+
                               |     数控振荡器 (NCO/DCO)     |
                               | (DPLL的核心，被反馈环路锁定) |
                               +--------------+--------------+
                                              |
                     +------------------------+------------------------+
                     |                                                 |
                     v                                                 v
           +-------------------+                           +-------------------------------+
           |  高频率时钟输出   |                           |    数字分频器 / 计数器       |
           | (e.g., 10 MHz)    |                           | (e.g., Divide by 10,000,000)   |
           +-------------------+                           +---------------+---------------+
                                                                         |
                                                                         v
                                                               +-------------------+
                                                               |   1PPS 输出信号    |
                                                               +-------------------+
关键点：
输出的 1PPS 信号之所以精确，完全是因为作为其源头的 NCO/DCO 的高频时钟已经被 DPLL 的反馈环路在相位上精确地锁定到了外部参考（如 GNSS 的 1PPS 输入）。这确保了分频后产生的脉冲不仅仅是“每秒一次”，而是“在每一秒精确的起点发生一次”。
```

## ts2phc是同步相位还是时钟？
```
ts2phc 这个工具主要用于同步相位和时间 (Phase and Time)。

虽然为了维持相位同步，它也必须调整时钟的频率，但其最终和核心的目标是相位对齐。

让我们来详细解释一下它的工作方式，您就能明白为什么它主要是同步相位。

ts2phc 的工作原理
ts2phc 的全称是 Timestamp to PTP Hardware Clock。它的典型应用场景是将一个网络接口卡（NIC）上的 PTP 硬件时钟 (PHC) 与一个外部的高精度时间源（通常是GNSS/GPS接收器）进行同步。

为此，ts2phc 需要从这个外部源获取两种信息：

1PPS (每秒脉冲) 信号: 这是一个极其精确的、标志着每一秒开始的物理脉冲信号。这是相位信息的来源。

时间日期 (Time of Day, ToD) 数据: 这是通过串口 (e.g., /dev/ttyS0) 发送的数据流 (通常是 NMEA 协议)，其中包含了年月日、时分秒等信息。这是时间信息的来源。

ts2phc 的同步过程如下：

读取时间: 它首先通过串口读取ToD数据，知道了“下一秒”将是具体哪个时间（例如，下午2点30分15秒）。

等待脉冲: 然后，它等待1PPS信号的到来。

捕获时间戳: 当1PPS脉冲到达的瞬间，ts2phc 会立即记录下目标PHC（网卡时钟）的当前时间戳。

计算误差: 它将“真实时间”（从ToD数据得知）与它刚刚捕获的PHC时间戳进行比较，计算出两者之间的偏移量 (offset)。

进行校准: ts2phc 会将这个偏移量信息反馈给内核，内核中的时钟伺服（clock servo）会不断地、平滑地调整PHC的相位和频率，以消除这个偏移量。

相位 vs. 时钟 (频率)
同步相位 (Phase Sync): 这是 ts2phc 的主要目标。它要确保PHC时钟计数器在 “下午2点30分15秒” 到来时的那个瞬间，与GNSS接收器的1PPS脉冲是完全对齐的。这个“对齐”就是相位同步。

同步时钟/频率 (Clock/Frequency Sync): 这是实现相位同步的必要手段。如果PHC的振荡器频率比标准秒稍微快一点或慢一点，那么即使在这一秒对齐了，下一秒它的相位也必然会再次产生偏差。因此，ts2phc 的伺服算法必须持续微调PHC的频率，使其不多不少，正好是标准的“每秒走一秒”的速度。

结论
ts2phc 是一个时间同步工具，其核心是相位同步。

它通过不断地将PHC的相位与高精度的1PPS参考源进行比对和校准来实现这一目标。虽然在这个过程中它也必须同步时钟的频率，但这只是维持长期相位稳定的前提条件。

所以，回答您的问题：ts2phc 主要是同步相位。
```

## SDP
```
SDP是连接物理接口和DPLL中间的软件逻辑，可以控制物理接口

What is an SDP (Software Defined Pin)?
SDP (Software Defined Pin / 软件定义引脚) 是 Intel E810 系列网卡上一项高级的硬件功能，它允许用户通过软件命令，动态地改变网卡上物理引脚（通常是 SMA 或 U.FL 接口）的功能、方向（输入/输出）以及信号类型。
The on-chip DPLL is the "brain" that performs the complex tasks of clock synchronization, filtering, and generation. The SDPs are the configurable "pipelines" that connect external real-world signals to and from this brain.

Summary Diagram
This diagram shows the relationship between the physical pins, the SDP configuration layer, and the DPLL engine.

                                      +--------------------------------+
                                      |         Intel E810 NIC         |
                                      |                                |
  External Signal <------> Physical Pin <------> SDP Configuration <------> +----------------+
  (e.g., GNSS 1PPS In,    (SMA / U.FL)   (Direction/Type/etc.)          | On-Chip DPLL   |
  10 MHz Out)                                                          | (The "Brain")  |
                                                                       +-------+--------+
                                                                               |
                                                                               v
                                                                      PTP & SyncE Clocks
                                                                      (from Network Port)


---- "Basically, the GPIOs for GNSS, SMA1, and U.FL1 are tied to connected SDPs..."
This means the physical interfaces (the onboard GNSS and the external SMA1/U.FL1 connectors) are not hardwired to one function. Instead, they are routed into the flexible Software Defined Pin (SDP) system, which acts like a smart switch or multiplexer.

---- "...if you configure SMA1 as an input pin (external timestamp), then the GNSS has to be disabled because it shares the same pin."
This describes the resource conflict. When you tell the software, "I want to use the SMA1 connector as my input," the system knows it shares the physical or logical "pin" (the input channel) with the GNSS. To prevent a signal collision, it automatically disables the GNSS module's input.

---- "But SMA2 doesn't share pins with GNSS, so it doesn't need to be automatically disabled."
This clarifies that the SMA2 connector has its own independent path to the timing chip. Configuring it as an input creates no conflict with the GNSS, so both can be configured without interference (though the DPLL would still need to be told which one to actively listen to).

Here is a simple diagram of the resource sharing:

[GNSS Module] ---+
                 |--> [Shared Internal Pin/Switch] ---> [DPLL Engine]
[SMA1 / U.FL1] --+

[SMA2] -----------> [Dedicated Internal Pin] -----> [DPLL Engine]


# https://issues.redhat.com/browse/RHEL-29207?focusedId=26697813&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-26697813
# https://issues.redhat.com/browse/RHEL-29207?focusedId=26531334&page=com.atlassian.jira.plugin.system.issuetabpanels%3Acomment-tabpanel#comment-26531334
cat > : <<- EOF
Basically, the GPIOs for GNSS, SMA1, and U.FL1 are tied to connected SDPs, if you configure SMA1 as an input pin (external timestamp), 
then the GNSS has to be disabled because it shares the same pin. 
But SMA2 doesn't share pins with GNSS, so GNSS doesn't need to be automatically disabled when enabling SMA2 input.

echo "1 0 0 0 100" > .../period which is equivalent to the PTP_PEROUT_REQUEST ioctl with a channel of 1, a  start time of 0 (as soon as possible) and a period of 100 nanoseconds.
Thi will fails with -EIO error code initially. 
This is because channel 1 of the output function is not assigned yet.(need to assign channel 1 of output function to a pin first)

You have to assign the pin to the correct function and channel
echo "2 1" > /sys/class/ptp/ptpN/pins/<name>

Here, the values are first the function type, 0 for disabled, 1 for external timestamp, 2 for periodic output, and 1 is the channel number of the function to assign. 
This informs the kernel you would like the given function on the pin.

That means, before enabling output on a channel, you must assign the output funciton of this channel(or channel of the output function?) to a pin first.
EOF

# https://issues.redhat.com/browse/RHEL-29207?focusedId=26225244&page=com.atlassian.jira.plugin.system.issuetabpanels:comment-tabpanel#comment-26225244
#   GNSS pin is on a different channel and on a different SDP, so there's no need to disable this one automatically (GNSS is on SDP21, SMA2 is on SDP23).

#   SDP20 is on channel1, SMA1 and U.FL1 are on channel1 , GNSS on channel1
#   SDP22 is on channel2, SMA2 and U.FL2 are on channel2

我理解是，
GNSS，SMA1，U.FL1，SDP20都在一个SDP通道上(channel，应该就是上面逻辑图中所说的SDP configuration layer）
SMA2, U.FL2, SDP22在另一个SDP通道上。
在一个通道上，同时只能有一个输入或输出。

但是这里面说的GNSS可能又不是GNSS物理输入接口。。。。因为GNSS和SDP20可以同时开启。。。
难道指的是SDP21吗？比如：
> 3. dpll still get locked when GNSS is disabled
This is not disabling GNSS, this is in fact disabling SDP21 input to CVL core. 
There is no way to actually disable GNSS from this API. 
I don't understand why is it designed like this, so I'm not able to explain it

```

## GPIO
```
GPIO is the acronym for General-Purpose Input/Output. In Chinese, it's called 通用输入/输出 (tōng yòng shū rù / shū chū).

It refers to a type of pin on an integrated circuit (like a microprocessor) or a circuit board (like a Raspberry Pi) whose function is not predefined and can be controlled by software at runtime.

Think of it as a programmable, digital switch that can be used for a wide variety of tasks.
```

##gnr-d 主要同步方法
```
https://docs.google.com/presentation/d/17jn4TmebZDlvrh293WJW1xwbuV7VnPyd/edit?slide=id.p1#slide=id.p1

2. GNR-D always powers up fed by the local crystal (XTAL_IN/OUT)
3. 使用ts2phc在NAC0和NAC1之间进行相位同步
4. select_clock_ref ()  Running PHC Clocks from DPLL Input
5. select_phy_ref () Running Ethernet PHY clocks from DPLL Input
6. 虽然PHC driven by DPLL input了，但是相位没有同步，可以使用ts2phc进行相位同步，也就是 “Synchronizing PHC to DPLL”
7. Synchronizing to a GNSS Module：PHY PHC driven by DPLL, DPLL driven by GNSS, PHC synced to DPLL with ts2phc
8. 同上，只不过DPLL driven by SMA
9. Using SyncE (Synchronous Ethernet) to Drive the DPLL: DPLL driven by SyncE, PHY and PHC still driven by DPLL, and PHC still synced to DPLL(ts2pc 1PPS)
   The select_recovered_clk () API can be used to select up to two ports per NAC that can be used to recover a frequency reference via SyncE
10. Using PTP (Precision Time Protocol) as the Clocking Reference: PHC同步到ptp4l之后，发送1PPS给DPLL，DPLL同步到这个1PPS，并且依然发送clock信号给PHC和PHY，PHC和PHY依然driven by DPLL
   When PHC0 becomes a clock master（这块应该是slave吧？） in ptp4l, the system needs to shift to drive a 1PPS into the DPLL via ETH01_SDP_TIMESYNC0
   If this is the highest priority input to the DPLL, this then drives all the outputs to be synchronous with the time that is derived from PTP


```

## gnr-d output pin phase-adjust
```
problem 4: the output pins' phase-adjust can't be changed (input pin works well on this)
Output pin phase adjust value has to be multiple of connected synthesizer half clock cycles.

In simple terms, the hardware that generates the output signal can only shift its phase in specific, discrete increments, like moving a Lego brick from one stud to the next. You can't place it halfway between two studs. For your hardware, that minimum increment is a "half clock cycle" of its internal synthesizer.


The output pin's phase adjustment is the same. 
It has a minimum "step size." You can adjust the phase by 1 step, 2 steps, or 3 steps, but not by 1.5 steps. For your hardware, that "step" is the duration of a half clock cycle of the synthesizer.

Practical Example
Let's say the synthesizer inside your DPLL runs at 100 MHz.

Clock Period: The period of a 100 MHz clock is 1 / 100,000,000 seconds = 10 nanoseconds.

Half Clock Cycle: Half of this period is 5 nanoseconds.

The Rule: This means your phase-adjust value for an output pin must be a multiple of 5 nanoseconds.

You can set phase-adjust to 0, 5, 10, 15, etc.

You cannot set phase-adjust to 3, 8, or 12. If you try, the hardware will likely reject the value or round it to the nearest valid multiple.


It depends on synthesizer's frequency connected to the particular output.

for GNR-D rework (05 and 06) there are 2 frequencies used for synths... 200MHz and 312.5MHz
then:
half_synth_cycle = 10^12 / (2 * synth_freq)
then
phase_adjust % half_synth_cycle == 0

for 312.5MHz: half_synth_cycle = 1600
for 200MHz: half_synth_cycle = 2500
so for outputs connected to synth with 312.5 the phase_adjust value has to be multiple of 1600
and for 200MHz has to be multiple of 2500

1600 * 25 = 40000
2500 * 16 = 40000

40000 value is 40000 ps -> 40 ns

```

## clock bandwidth
```
The main metric used to describe how noise is passed from input to output is clock bandwidth, which was explained in the section “Low-Pass and High-Pass Filters.
For example, the noise transfer of an Option 1 EEC is described in clause 10.1 of ITU-T G.8262 as follows: 
“The minimum bandwidth requirement for an EEC is 1 Hz. The maximum bandwidth requirement for an EEC is 10 Hz.”


Narrow Bandwidth (e.g., 1 Hz) is like a soft, floaty suspension on a luxury car. 
It's very slow to react and absorbs almost all the small, fast bumps (high-frequency jitter), giving you a very smooth ride.

Wide Bandwidth (e.g., 10 Hz) is like a stiff, sporty suspension on a race car. 
It's very quick to react and follows the road surface closely. You feel more of the small bumps (lets more jitter through), 
but it responds instantly to actual turns (tracks the input signal's wander).

```


## Stratum Clock System
```
This is a hierarchy used in telecommunications to rank timing sources by their accuracy. The lower the stratum number, the higher the accuracy.

Stratum 1: The highest level. A completely autonomous source like a Cesium atomic clock or a GNSS (GPS) receiver. It does not need to be synchronized to anything else.

Stratum 2: These clocks, like one based on a Rubidium oscillator, are very stable and are synchronized to a Stratum 1 source. They have excellent holdover capabilities.

Stratum 3: These clocks are synchronized to a Stratum 2 source. A high-quality OCXO (Oven-Controlled Crystal Oscillator) is a typical Stratum 3 clock.

Stratum 3E: An "enhanced" version of Stratum 3 with much better stability and holdover.

Stratum 4: A much lower-quality clock, often a simple, uncompensated crystal oscillator (XO). This is the type of clock found in most standard network equipment like routers and switches. Its holdover performance is very poor.

```

# 1 agenda

- TimeKeeping, how NTP  works
- RoundTrip, measure clock error between hosts
- UDP vs TCP from programming prospective


# 2 the science of time keeping


## 2.1 what is clock
时间是日常生活中测量精度最高的物理量了,通常能达到10个有效数字,即 `86400*365*70=(24*3600)*365*70` 即一个人从出生到死亡的任何一秒,

- 什么是一个时钟(clock)? 
	 $A\space clock = An\space oscillator + a\space counter$

**振荡器(oscillator)** 提供一个稳定、重复的信号。它可以是来回摆动的钟摆、振动的石英晶体，甚至是原子钟中原子的振荡。振荡器是时钟的“心脏”。

**计数器** 记录振荡的次数。它计算振荡器完成一个周期的次数。然后用这个计数来表示时间单位，比如秒、分和小时。

举个简单的例子：

想象一个来回摆动的钟摆。每次摆动（一个完整周期）需要一秒钟。钟摆就是振荡器。一个记录每次摆动的装置（1 次摆动 = 1 秒，2 次摆动 = 2 秒，以此类推）就是计数器。它们共同构成了一个时钟。


所以当前时间计算为:
$$current\space time=count/frequency +offset$$
即计数值除以频率 + 一个起止时间的偏置, 可以看出一个时间的 `offset`可以准确的算出或设置, `oscillator` 的质量, 它频率的`准确度`和`稳定度`,决定了时钟的好坏

我们电子电路常用的是 `石英晶体振荡器(quarts crystal oscillators)`简称 `晶振`


- 常见时钟的晶振准确度如图
 ![[{AE9933B3-CAC1-4CE6-9C2F-194CD0680F99}.png]]
 一个 `ppm(parts in per milion`即$10^{-6}$ , 一天 `86400`s , `Clock XO` 误差按 `10ppm/s`  , 一天即 `864ms=0.8s` 误差, 


- 在 `linux kernel`对时间测量如下
![[{1CD132D6-733E-4853-B841-72CBD7D7EB64}.png]]
## 2.2 时钟的准确度和校准


- 为什么出厂时不直接测准晶振的频率,然后再记录下来用呢?
	因为晶振的frequency 会随着 `temperature` 和 `aging` 的改变而改变


我们一般需求如下晶振
![[{6CD9F244-F0A5-4B66-BD95-3F955C80B041}.png]]
 
 
 测量时间时, 我们一般测量 `timestamp` 和 `Time interval`,其满足的运算可抽象为$$tp = tp + ti, ti=ti+-ti,ti=tp-tp$$, 即$tp+tp = tp$ 不合法,要求两个 `tp` 之间的中间时间点$\frac{tp_1+tp_2}{2}$, 可通过$$tp_{mid}= tp1 + \frac{tp2-tp1}{2}$$
 在 `linux kernel` 我们对时间的测量如下图
## 2.3 网络时间的同步(NTP)
> `Network Time Protocol（NTP）`是用来使计算机时间同步化的一种协议，它可以使计算机对其服务器或时钟源（如石英钟，GPS等等)做同步化，它可以提供高精准度的时间校正（LAN上与标准间差小于1毫秒，WAN上几十毫秒），且可介由加密确认的方式来防止恶毒的协议攻击。NTP的目的是在无序的Internet环境中提供精确和健壮的时间服务

[NTP原理及工作模式]([NTP的工作原理以及工作模式 - 知乎](https://zhuanlan.zhihu.com/p/106069365#:~:text=NTP%EF%BC%88Network%20Time%20Protocol%EF%BC%8C%E7%BD%91%E7%BB%9C%E6%97%B6%E9%97%B4%E5%8D%8F%E8%AE%AE%EF%BC%89%E6%98%AF%E7%94%B1RFC%201305%E5%AE%9A%E4%B9%89%E7%9A%84%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5%E5%8D%8F%E8%AE%AE%EF%BC%8C%E7%94%A8%E6%9D%A5%E5%9C%A8%E5%88%86%E5%B8%83%E5%BC%8F%E6%97%B6%E9%97%B4%E6%9C%8D%E5%8A%A1%E5%99%A8%E5%92%8C%E5%AE%A2%E6%88%B7%E7%AB%AF%E4%B9%8B%E9%97%B4%E8%BF%9B%E8%A1%8C%E6%97%B6%E9%97%B4%E5%90%8C%E6%AD%A5%E3%80%82.%20NTP%E5%9F%BA%E4%BA%8EUDP%E6%8A%A5%E6%96%87%E8%BF%9B%E8%A1%8C%E4%BC%A0%E8%BE%93%EF%BC%8C%E4%BD%BF%E7%94%A8%E7%9A%84UDP%E7%AB%AF%E5%8F%A3%E5%8F%B7%E4%B8%BA123%E3%80%82.%20%E4%BD%BF%E7%94%A8NTP%E7%9A%84%E7%9B%AE%E7%9A%84%E6%98%AF%E5%AF%B9%E7%BD%91%E7%BB%9C%E5%86%85%E6%89%80%E6%9C%89%E5%85%B7%E6%9C%89%E6%97%B6%E9%92%9F%E7%9A%84%E8%AE%BE%E5%A4%87%E8%BF%9B%E8%A1%8C%E6%97%B6%E9%92%9F%E5%90%8C%E6%AD%A5%EF%BC%8C%E4%BD%BF%E7%BD%91%E7%BB%9C%E5%86%85%E6%89%80%E6%9C%89%E8%AE%BE%E5%A4%87%E7%9A%84%E6%97%B6%E9%92%9F%E4%BF%9D%E6%8C%81%E4%B8%80%E8%87%B4%EF%BC%8C%E4%BB%8E%E8%80%8C%E4%BD%BF%E8%AE%BE%E5%A4%87%E8%83%BD%E5%A4%9F%E6%8F%90%E4%BE%9B%E5%9F%BA%E4%BA%8E%E7%BB%9F%E4%B8%80%E6%97%B6%E9%97%B4%E7%9A%84%E5%A4%9A%E7%A7%8D%E5%BA%94%E7%94%A8%E3%80%82.%20%E5%AF%B9%E4%BA%8E%E8%BF%90%E8%A1%8CNTP%E7%9A%84%E6%9C%AC%E5%9C%B0%E7%B3%BB%E7%BB%9F%EF%BC%8C%E6%97%A2%E5%8F%AF%E4%BB%A5%E6%8E%A5%E6%94%B6%E6%9D%A5%E8%87%AA%E5%85%B6%E4%BB%96%E6%97%B6%E9%92%9F%E6%BA%90%E7%9A%84%E5%90%8C%E6%AD%A5%EF%BC%8C%E5%8F%88%E5%8F%AF%E4%BB%A5%E4%BD%9C%E4%B8%BA%E6%97%B6%E9%92%9F%E6%BA%90%E5%90%8C%E6%AD%A5%E5%85%B6%E4%BB%96%E7%9A%84%E6%97%B6%E9%92%9F%EF%BC%8C%E5%B9%B6%E4%B8%94%E5%8F%AF%E4%BB%A5%E5%92%8C%E5%85%B6%E4%BB%96%E8%AE%BE%E5%A4%87%E4%BA%92%E7%9B%B8%E5%90%8C%E6%AD%A5%E3%80%82.,NTP%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.%20NTP%E7%9A%84%E5%9F%BA%E6%9C%AC%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86%E5%A6%82%E5%9B%BE%E6%89%80%E7%A4%BA%E3%80%82.%20Device%20A%E5%92%8CDevice%20B%E9%80%9A%E8%BF%87%E7%BD%91%E7%BB%9C%E7%9B%B8%E8%BF%9E%EF%BC%8C%E5%AE%83%E4%BB%AC%E9%83%BD%E6%9C%89%E8%87%AA%E5%B7%B1%E7%8B%AC%E7%AB%8B%E7%9A%84%E7%B3%BB%E7%BB%9F%E6%97%B6%E9%92%9F%EF%BC%8C%E9%9C%80%E8%A6%81%E9%80%9A%E8%BF%87NTP%E5%AE%9E%E7%8E%B0%E5%90%84%E8%87%AA%E7%B3%BB%E7%BB%9F%E6%97%B6%E9%92%9F%E7%9A%84%E8%87%AA%E5%8A%A8%E5%90%8C%E6%AD%A5%E3%80%82.%20%E4%B8%BA%E4%BE%BF%E4%BA%8E%E7%90%86%E8%A7%A3%EF%BC%8C%E4%BD%9C%E5%A6%82%E4%B8%8B%E5%81%87%E8%AE%BE%EF%BC%9A.%20))

测量两台时间差的公式为$$\frac{(T_4+T_1)-(T_{2}+T_3)}{2}$$,解释如下图
![[{EA4120AC-7C7D-4FEB-B671-82EF2902F88B}.png]]

- why do we need to know clock error?
	1. 测量NTP是否工作
	2. Timestamp from two clocks(not in the same clock domain) are not comparable unless you know the offset
	![[{39BB5E3F-624A-4EAC-9D0B-841CA0081D22}.png]]

# 3 code roundtrip 

## 3.1 udp
>我们仿照NTP时间同步的原理，实现一个测量两台机器之间误差的程序，在实现中服务端将收到数据的时间与发送应答的时间抽象为一个时间点，即忽略server端处理数据的机器误差。利用这三个时间点 T1, T2, T3，从而计算两台机器的时延。



实验代码如下
- UDP, two threads
	[roundtrip_udp.cc](https://github.com/CottonTnT/recipes/blob/master/tpc/bin/roundtrip_udp.cc)

- -UDP with muduo, single thread  
    [muduo/examples/roundtrip/roundtrip_udp.cc](https://gitee.com/TRr320/muduo-master/blob/master/examples/roundtrip/roundtrip_udp.cc)
- TCP with muduo  
    [muduo/examples/roundtrip/roundtrip.cc](https://gitee.com/TRr320/muduo-master/blob/master/examples/roundtrip/roundtrip.cc)
	![[{D358A6BC-B224-4633-804E-F763F34DCB72}.png]]


# 4 	UDP vs TCP
TCP：传输控制协议。面向连接的、可靠的、基于字节流的传输层通信协议。特点：可靠传输（应答确认）、提供拥塞控制、全双工通信（允许通信双方的应用程序在任何时候都能发送数据，因为TCP连接的两端都设有缓存，用来临时存放双向通信的数据）。

UDP：用户数据报协议。无连接的、 不可靠的、 面向数据报的协议。特点：提供单播，多播，广播的功能

- UDP的使用场景（一般情况下尽量使用TCP协议）：
	- 资源受限的情况下：例如 nat穿透技术。在互联网技术中，UDP常用在缓存读取，保存；用在监控或终端上报。
	- 不在乎可靠性：例如 本节的 round_trip 程序，消息本身不需要重传，使用udp足够应对。又例如在 直播、视屏通话等场景中，需要的是实时性，如果因重传而造成延迟卡顿，那么用户的使用体验将会很糟糕。


- 网络编程角度差别：
	- TCP编程：1. 并发编程需要为每个客户端都创建socket连接，为每个客户端都配备文件描述符。同时需要考虑线程间的分配问题。 2. TCP线程不安全，应避免多个线程同时读取一个文件描述符。
	
	- UDP编程：1. 并发编程中，服务端一个socket可以服务多个客户端。2. 线程安全，数据报协议自动维护消息的边界，多线程操作同一个描述符也不会出现粘包，半包的问题（每次从接收缓冲都是只能取回一个完整的包）。

	使用UDP编程，需要自己确保可靠性（参考tcp实现，设计心跳机制、应答确认、超时重传等）。 注：UDP协议栈会在发送的时候分片，到了接收重组，每次组好了完整的包就丢给用户，不完整就丢掉，故，可能会出现丢包现象。

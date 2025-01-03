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




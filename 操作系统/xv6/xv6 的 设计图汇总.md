# 1 Risc-v MMU theory 


图 3.2 显示了这一切的工作原理。标志位和与分页硬件相关的数据结构定义在（`kernel/riscv.h`）中。

![img](assets/figure-3.2.jpg)

*要告诉硬件使用一个页表，内核必须将对应**根页表页的物理地址**写入 `satp` 寄存器中*。每个 `CPU` 都有自己的 `satp` 寄存器。一个 `CPU` 将使用自己的 `satp` 所指向的页表来翻译后续指令产生的所有地址。每个 `CPU` 都有自己的 `satp`，这样不同的 `CPU` 可以运行不同的进程，每个进程都有自己的页表所描述的私有地址空间。




# 2 kernel pagetable mapping
图 3.3 显示了这个设计是如何将内核虚拟地址映射到物理地址的。

 ![img](assets/Figure-3.3.jpg) 
# 3 user pagetable mapping


![img](assets/Figure-3.4.jpg)
图 3.4 更详细地显示了 `xv6` 中执行进程的用户内存布局。*栈只有一页*，图中显示的是由`exec` 创建的初始内容。位于栈顶部的字符串中包含了*命令行中输入的参数*和*指向他们的指针数组*。在下方是允许程序在 `main` 启动的值，就像函数 `main(argc, argv)` 是刚刚被调用一样。
# Untitled

RISC-V有对于中断的许多支持。首先是一些中断相关的寄存器：

* SIE（Supervisor Interrupt Enable）寄存器。这个寄存器中有一个bit专门针对外部设备（E），例如UART，的中断；有一个bit专门针对软件中断（S），软件中断可能又一个CPU核触发发给另一个CPU核；还有一个bit专门针对时间中断（T）。我们这里只关注外部设备的中断。
* SSTATUS（Supervisor Status）寄存器。这个寄存器中有一个bit来打开或者关闭中断。每一个CPU核都有这两个寄存器，除了能够单独控制特定的中断，在SSTATUS寄存器中还有一个bit能够控制所有的中断。
* SIP（Supervisor Interrupt Pending）寄存器。当发生中断时，处理器可以通过查看这个寄存器知道当前是什么类型的中断。
* SCAUSE寄存器，这个寄存器我们之前看过很多次。它会表明当前状态的原因是中断。
* STVEC寄存器，它会保存当trap，page fault或者中断发生时，CPU运行的用户程序的程序计数器。因为在这3个场景下，XV6使用的是相同的处理流程。

我们今天不会怎么讨论SCAUSE和STVEC寄存器，因为我们之前已经讨论过（注，lec06），在中断处理流程中，它们基本上与之前的工作方式是一样的。接下来我们看看XV6是如何对这些寄存器进行编程，使得它们处于一个能接受中断的状态。

接下来看看代码，首先是位于start.c的start函数。

![](../.gitbook/assets/image%20%28369%29.png)

这里将所有的中断都设置在Supervisor mode，然后设置SIE寄存器来接收External，软件和定时器中断，之后初始化定时器。

接下来我们看一下main函数中是如何处理External中断。

![](../.gitbook/assets/image%20%28368%29.png)

我们第一个外设是console，这是我们print的输出位置。查看位于console.c的consoleinit函数。

![](../.gitbook/assets/image%20%28371%29.png)

这里首先初始化了锁，我们现在还不关心这个锁。然后调用了uartinit。uartinit函数位于uart.c文件。这个函数实际上就是配置好UART芯片使其可以被使用。

![](../.gitbook/assets/image%20%28372%29.png)


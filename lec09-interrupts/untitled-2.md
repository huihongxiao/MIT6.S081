# 9.5 XV6设置中断

RISC-V有对于中断的许多支持。首先是一些中断相关的寄存器：

* SIE（Supervisor Interrupt Enable）寄存器。这个寄存器中有一个bit专门针对外部设备（E），例如UART，的中断；有一个bit专门针对软件中断（S），软件中断可能又一个CPU核触发发给另一个CPU核；还有一个bit专门针对时间中断（T）。我们这里只关注外部设备的中断。
* SSTATUS（Supervisor Status）寄存器。这个寄存器中有一个bit来打开或者关闭中断。每一个CPU核都有这两个寄存器，除了能够单独控制特定的中断，在SSTATUS寄存器中还有一个bit能够控制所有的中断。
* SIP（Supervisor Interrupt Pending）寄存器。当发生中断时，处理器可以通过查看这个寄存器知道当前是什么类型的中断。
* SCAUSE寄存器，这个寄存器我们之前看过很多次。它会表明当前状态的原因是中断。
* STVEC寄存器，它会保存当trap，page fault或者中断发生时，CPU运行的用户程序的程序计数器。因为在这3个场景下，XV6使用的是相同的处理流程。

我们今天不会怎么讨论SCAUSE和STVEC寄存器，因为我们之前已经讨论过（注，lec06），在中断处理流程中，它们基本上与之前的工作方式是一样的。接下来我们看看XV6是如何对这些寄存器进行编程，使得它们处于一个能接受中断的状态。

接下来看看代码，首先是位于start.c的start函数。

![](../.gitbook/assets/image%20%28378%29.png)

这里将所有的中断都设置在Supervisor mode，然后设置SIE寄存器来接收External，软件和定时器中断，之后初始化定时器。

接下来我们看一下main函数中是如何处理External中断。

![](../.gitbook/assets/image%20%28373%29.png)

我们第一个外设是console，这是我们print的输出位置。查看位于console.c的consoleinit函数。

![](../.gitbook/assets/image%20%28387%29.png)

这里首先初始化了锁，我们现在还不关心这个锁。然后调用了uartinit。uartinit函数位于uart.c文件。这个函数实际上就是配置好UART芯片使其可以被使用。

![](../.gitbook/assets/image%20%28388%29.png)

这里的流程是先关闭中断，之后设置波特率，设置字符长度为8bit，重置FIFO，最后再重新打开中断。

> 学生提问：什么是波特率？
>
> Frans教授：这是串口线的传输速率。

以上就是uartinit函数，运行完这个函数之后，原则上UART就可以生成中断了。但是因为我们还没有对PLIC编程，所以中断不能被CPU感知。最终，在main函数中，需要调用plicinit函数。下图是plicinit函数。

![](../.gitbook/assets/image%20%28377%29.png)

PLIC与外设一样，也占用了一个I/O地址（0xC000\_0000）。代码的第一行使能了UART的中断，这里实际上就是设置PLIC会接收哪些中断，进而将中断路由到CPU。类似的，代码的第二行设置PLIC接收来自IO磁盘的中断，我们这节课不会介绍这部分内容。

main函数中，plicinit之后就是plicinithart函数。plicinit是由0号CPU运行，之后，每个CPU的核都需要表明对于哪些外设中断感兴趣。

![](../.gitbook/assets/image%20%28374%29.png)

所以在plicinithart函数中，每个CPU的核都表明自己对来自于UART和VIRTIO的中断感兴趣。因为我们忽略中断的优先级，所以我们将优先级设置为0。

到目前为止，我们有了生成中断的设备，我们有了PLIC可以传递中断到单个的CPU。但是CPU自己还没有设置好接收中断，因为我们还没有设置好SSTATUS寄存器。在main函数的最后，程序调用了scheduler函数，

![](../.gitbook/assets/image%20%28386%29.png)

scheduler函数主要是运行进程。但是在实际运行进程之前，会执行intr\_on函数来使得CPU能接收中断。

![](../.gitbook/assets/image%20%28390%29.png)

intr\_on函数只完成一件事情，就是设置SSTATUS寄存器，打开中断标志位。

在这个时间点，中断被完全打开了。如果PLIC正好有pending的中断，那么这个CPU核会收到中断。

以上就是中断的基本设置。

> 学生提问：哪些核在intr\_on之后打开了中断？
>
> Frans教授：任何一个调用了intr\_on的CPU核，都会接收中断。实际上所有的CPU核都会运行intr\_on函数。




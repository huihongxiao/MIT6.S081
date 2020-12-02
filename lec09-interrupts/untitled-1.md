# Untitled

通常来说，管理设备的代码称为驱动，所有的驱动都在内核中。我们今天要看的代码在uart.c中，这个是UART设备的驱动。如果我们查看代码的结构，我们可以发现大部分驱动都分为两个部分，bottom/top。

bottom部分通常是Interrupt handler。当一个中断送到了CPU，并且CPU打开了这个中断，CPU会调用相应的Interrupt handler。Interrupt handler并不运行在任何特定进程的context中，它只是处理中断。

top部分，是用户进程，或者内核的其他部分调用的接口。对于UART来说，这里有read/write接口，这些接口可以被更高层级的代码调用。

![](../.gitbook/assets/image%20%28358%29.png)

通常情况下，驱动中会有一些队列，top部分的代码会从队列中读写数据，而Interrupt handler（bottom部分）同时也会向队列中读写数据。当设备与CPU并行运行时，这里的队列将top/bottom部分解耦开来。

![](../.gitbook/assets/image%20%28359%29.png)

对于Interrupt handler，存在一些限制。因为它并没有运行在任何context中，所以它不能调用copy in/out，因为kernel page table并不知道应该copy哪个字符。所以驱动的top部分通常与用户层面的进程交互，并进行copy in/out。我们后面会看更多的细节，但是这里是一个驱动的典型架构。

在很多操作系统中，驱动代码加起来可能会比内核还要大，主要是因为，对于每个设备，你都需要一个驱动，而设备又很多。



接下来我们看一下如何对设备进行编程。通常来说，编程是通过memory mapped I/O完成的。在SiFive的手册中，设备地址出现在物理地址的特定区间内。这个区间由主板制造商决定。操作系统需要知道这些设备位于物理地址空间的具体位置，然后再通过普通的load/store指令对这些地址进行编程。


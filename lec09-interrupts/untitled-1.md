# 9.3 设备驱动概述

通常来说，管理设备的代码称为驱动，所有的驱动都在内核中。我们今天要看的是UART设备的驱动，代码在uart.c文件中。如果我们查看代码的结构，我们可以发现大部分驱动都分为两个部分，bottom/top。

bottom部分通常是Interrupt handler。当一个中断送到了CPU，并且CPU设置接收这个中断，CPU会调用相应的Interrupt handler。Interrupt handler并不运行在任何特定进程的context中，它只是处理中断。

top部分，是用户进程，或者内核的其他部分调用的接口。对于UART来说，这里有read/write接口，这些接口可以被更高层级的代码调用。

![](../.gitbook/assets/image%20%28358%29.png)

通常情况下，驱动中会有一些队列（或者说buffer），top部分的代码会从队列中读写数据，而Interrupt handler（bottom部分）同时也会向队列中读写数据。这里的队列可以将并行运行的设备和CPU解耦开来。

![](../.gitbook/assets/image%20%28359%29.png)

通常对于Interrupt handler来说存在一些限制，因为它并没有运行在任何context中，所以不能直接从它读写数据。因为kernel page table并不知道应该copy哪个字符。所以驱动的top部分通常与用户层面的进程交互，并进行copy in/out。我们后面会看更多的细节，但是这里是一个驱动的典型架构。

在很多操作系统中，驱动代码加起来可能会比内核还要大，主要是因为，对于每个设备，你都需要一个驱动，而设备又很多。




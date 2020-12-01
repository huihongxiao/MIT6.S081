# Untitled

通常来说，管理设备的代码称为驱动，所有的驱动都在内核中。我们今天要看的代码在uart.c中，这个是UART设备的驱动。如果我们查看代码的结构，我们可以发现大部分驱动都分为两个部分，bottom/top。

bottom部分通常是Interrupt handler。当一个中断送到了CPU，并且CPU打开了这个中断，CPU会调用相应的Interrupt handler。Interrupt handler并不运行在任何特定进程的context中，它只是处理中断。

top部分，是用户进程，或者内核的其他部分调用的接口。对于UART来说，这里有read/write接口，这些接口可以被更高层级的代码调用。

![](../.gitbook/assets/image%20%28358%29.png)

通常情况下，驱动中会有一些队列，top部分的代码会从队列中读写数据，而Interrupt handler（bottom部分）同时也会向队列中读写数据。

![](../.gitbook/assets/image%20%28359%29.png)


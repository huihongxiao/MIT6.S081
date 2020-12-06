# 9.7 UART将字符输出到Console

在我们向console输出字符时，如果发生了中断，RISC-V会做什么操作？我们之前已经在SSTATUS寄存器中打开了中断，所以处理器会被中断。假设键盘生成了一个中断并且发向了PLIC，PLIC将中断路由给了我一个特定的CPU核，并且这个CPU核设置了SIE寄存器的bit（注，针对外部中断的E bit位），那么会发生以下事情：

* 首先，程序会清除SIE寄存器相应的bit，这样可以阻止CPU核被其他中断打扰，该CPU核可以专心处理当前中断。处理完成之后，可以再次恢复SIE寄存器相应的bit。
* 之后，程序会设置SEPC寄存器为当前的程序计数器。我们假设Shell正在用户空间运行，突然来了一个中断，那么当前Shell的程序计数器会被保存。
* 之后，程序要保存当前的mode。在我们的例子里面，因为当前运行的是Shell程序，所以会记录user mode。1
* 再将mode设置为Supervisor mode。
* 最后将程序计数器的值设置成STVEC的值。（注，STVEC用来保存trap处理程序的地址，详见lec06）在XV6中，STVEC保存的要么是uservec或者kernelvec函数的地址，具体取决于发生中断时是在用户空间还是内核空间。在我们的例子中，Shell运行在用户空间，所以STVEC保存的是uservec函数的地址。而从之前的课程我们可以知道uservec函数会调用usertrap函数。所以最终，我们在usertrap函数中。我们这节课不会介绍trap过程中的拷贝，恢复过程，因为在之前的课程中已经详细的介绍过了。

![](../.gitbook/assets/image%20%28376%29.png)

接下来看一下trap.c文件中的usertrap函数，我们在lec06和lec08分别在这个函数中处理了系统调用和page fault。今天我们将要看一下如何处理中断。

![](../.gitbook/assets/image%20%28387%29.png)

在trap.c的devintr函数中，首先会通过SCAUSE寄存器判断当前中断是否是来自于外设的中断。如果是的话，再调用plic\_claim函数来获取中断。

![](../.gitbook/assets/image%20%28399%29.png)

![](../.gitbook/assets/image%20%28398%29.png)

plic\_claim函数位于plic.c文件中。在这个函数中，当前CPU核会告知PLIC，自己要获得中断，PLIC\_SCLAIM会将中断号返回，对于UART来说，返回的中断号是10。

![](../.gitbook/assets/image%20%28372%29.png)

从devintr函数可以看出，如果是UART中断，那么会调用uartintr函数。位于uart.c文件的uartintr函数，会从UART的接受寄存器中读取数据，之后将获取到的数据传递给consoleintr函数。哦，不好意思，我搞错了。我们现在讨论的是向UART发送数据。因为我们现在还没有通过键盘输入任何数据，所以UART的接受寄存器现在为空。

![](../.gitbook/assets/image%20%28385%29.png)

所以代码会直接运行到uartstart函数，这个函数会将Shell存储在buffer中的任意字符送出。实际上在提示符“$”之后，还有一个空格字符，write系统调用可以在UART发送提示符“$”的同时，并发的将空格字符写入。所以UART的发送中断触发时，可以发现在buffer中还有一个空格字符，之后会将这个空格字符送出。

> 学生提问： UART对于键盘来说很重要，来自于键盘的字符通过UART走到CPU再到我们写的代码。但是我不太理解UART对于Shell输出字符究竟有什么作用？因为在这个场景中，并没有键盘的参与。
>
> Frans教授：显示设备与UART也是相关的。所以UART连接了两个设备，一个是键盘，另一个是显示设备，也就是Console。QEMU也是通过模拟的UART与Console进行交互，而Console的作用就是将字符在显示器上画出来。


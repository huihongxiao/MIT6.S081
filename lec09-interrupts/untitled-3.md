# Untitled

接下来我想看一下如何从Shell程序输出提示符“$”。首先我们看init.c中的main函数，这是第一个运行的进程。

![](../.gitbook/assets/image%20%28368%29.png)

首先这个进程的main函数创建了一个代表console的设备。这里通过mknod操作创建了console设备。并且获得文件描述符0，因为这是第一个打开的文件。之后通过dup创建stdout和stderr。这里实际上创建了文件描述符0，1，2来代表console。

Shell程序首先打开文件描述符0，1，2。之后Shell向文件描述符2打印提示符“$ ”。

![](../.gitbook/assets/image%20%28380%29.png)

尽管console背后是UART设备，但是从应用程序来看，就像是一个普通的文件。Shell程序只是想文件描述符2写了数据，它并不知道文件描述符2对应的是什么。在Unix系统中，设备是由文件表示与其他东西并没有区别。我们来看一下这里的打印是如何工作的。

在printf.c文件中，代码只是调用了write系统调用，在我们的例子中，fd对应的就是文件描述符2。c是字符“$”。

![](../.gitbook/assets/image%20%28377%29.png)

所以由Shell输出的每一个字符都会引起一个write系统调用。之前我们已经看过了write系统调用最终会走到sysfile.c文件的sys\_write函数。

![](../.gitbook/assets/image%20%28366%29.png)

这里首先对参数做了检查，然后又调用了filewrite函数。filewrite函数位于file.c文件中。

![](../.gitbook/assets/image%20%28379%29.png)

在filewrite函数中首先会判断文件描述符的类型。mknod生成的文件描述符属于设备（FD\_DEVICE），而对于设备类型的文件描述符，我们会为这个特定的设备执行write函数。因为我们现在的设备是console，所以我们知道这里会调用console.c中的write函数。

![](../.gitbook/assets/image%20%28365%29.png)

这里先通过either\_copyin将字符拷入，之后调用uartputc函数。这里将字符写入和UART设备，所以你可以认为console是一个UART驱动的top部分。uart.c文件中的uartputc函数会实际的打印字符。

![](../.gitbook/assets/image%20%28384%29.png)

uartputc函数会稍微有趣一些。在UART的内部会有一个buffer用来发送数据，buffer的大小是32个字符。同时还有一个读指针和写指针，来构建一个环形的buffer。

![](../.gitbook/assets/image%20%28370%29.png)

在我们的例子中，使用的是写指针。


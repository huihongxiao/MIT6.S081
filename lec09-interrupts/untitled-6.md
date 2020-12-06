# Untitled

在UART的另一侧，会有类似的事情发生，有时Shell会调用read从键盘中读取字符。 在read系统调用的底层，会调用fileread函数。在这个函数中，如果读取的文件类型是设备，会调用相应设备的read函数。

![](../.gitbook/assets/image%20%28390%29.png)

在我们的例子中，就是console.c文件中的consoleread函数。

![](../.gitbook/assets/image%20%28395%29.png)

这里与UART类似，也有一个buffer，包含了128个字符。其他的基本一样，也有producer和consumser。但是在这个场景下Shell变成了consumser，因为Shell是从buffer中读取数据。而键盘是producer，它将数据写入到buffer中。

![](../.gitbook/assets/image%20%28380%29.png)

从consoleread函数中可以看出，当读指针和写指针一样时，说明buffer为空，进程会sleep。所以Shell在打印完“$ ”之后会sleep，直到有一个字符输入。所以在某个时间点，假设用户通过键盘输入了“l”，这会导致“l”被发送到主板上的UART芯片，产生中断之后再被PLIC路由到某个CPU核，之后会触发devintr函数，devintr可以发现这是一个UART中断，然后通过uartgetc函数获取到相应的字符，之后再将字符传递给consoleintr函数。

![](../.gitbook/assets/image%20%28383%29.png)


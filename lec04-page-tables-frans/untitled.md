# 4.6 kvminit

接下来，让我们看一看代码，我认为很多东西都会因此变得更加清晰。

首先，我们来做一个的常规操作，启动我们的XV6，这里QEMU实现了主板，同时我们打开gdb。

![](../.gitbook/assets/image%20%28191%29.png)

上一次我们看了boot的流程，我们跟到了main函数。main函数中调用的一个函数是kvminit（3.9），这个函数会设置好kernel的地址空间。我们在前一部分看了kernel的地址空间长成什么样，这里我们来看一下代码是如何将它设置好的。首先在kvminit中设置一个断点，之后运行代码到断点位置。kvminit的代码如下图所示：

![](../.gitbook/assets/image%20%28199%29.png)

在gdb中执行layout split，可以看到（从上面的代码也可以看出）函数的第一步是为最高一级page directory分配物理page（怎么看出来的？？？）。下一行将这段内存初始化为0。

![](../.gitbook/assets/image%20%28193%29.png)

之后，通过kvmmap函数，将每一个I/O设备映射到内核。例如，下图中高亮的行将UART0映射到内核的地址空间。

![](../.gitbook/assets/image%20%28195%29.png)

我们可以查看一个文件叫做memlayout.h，它将4.5中文档翻译成了一堆常量。在这个文件里面可以看到，UART0对应了地址0x10000000（注，4.5中的文档是真正SiFive RISC-V的文档，而下图是QEMU的地址，所以4.5中的文档地址与这里的不符）。

![](../.gitbook/assets/image%20%28194%29.png)

所以，通过kvmmap可以将物理地址映射到相同的虚拟地址（kvmmap的前两个参数一致）。

在page table实验中，第一个练习是实现vmprint，这个函数会打印当前的kernel page table。我们现在跳过这个函数，看一下执行完第一个kvmmap时的kernel page table。

![](../.gitbook/assets/image%20%28190%29.png)

我们来看一下这里的输出。第一行是最高一级page directory的地址，这就是存在SATP或者将会存在SATP的地址。第二行可以看到最高一级page directory只有一条PTE序号为0，它包含了中间级page directory的物理地址。第三行可以看到中间级的page directory只有一条PTE序号为128，它指向了最低级page directory的物理地址。第四行可以看到最低级的page directory包含了PTE指向物理地址。你们可以看到最低一级 page directory中PTE的物理地址就是0x10000000，对应了UART0。

前面是物理地址，我们可以从虚拟地址的角度来验证这里符合预期。我们将地址0x10000000向右移位12bit，这样可以得到虚拟地址的高27bit（index部分）。之后我们再对这部分右移位9bit，并打印成10进制数，可以得到128，这就是中间级page directory中PTE的序号。

![](../.gitbook/assets/image%20%28198%29.png)

这里与之前介绍的内容是符合的。

从标志位来看（fl部分），最低一级page directory中的PTE有读写标志位，并且Valid标志位也设置了（4.3底部有标志位的介绍）。

内核会持续的按照这种方式，调用kvmmap来设置地址空间。之后会对VIRTIO0、CLINT、PLIC、kernel text、kernel data、最后是TRAMPOLINE进行地址映射。最后我们还会调用vmprint打印完整的kernel page directory，可以看出已经设置了很多PTE。

![](../.gitbook/assets/image%20%28197%29.png)

这里就不过细节了，但是这些PTE构成了我们在4.5中看到的虚拟地址映射关系。

之后，kvminit函数返回了，在main函数中，我们运行到了kvminithart函数。

![](../.gitbook/assets/image%20%28196%29.png)

这个函数首先设置了SATP寄存器，kernel\_pagetable变量来自于kvminit第一行。所以这里内核告诉MMU来使用刚刚设置好的page table。当这里这条指令执行之后下一个地址被翻译的时候会发生什么？

在这条指令之前，还不存在可用的page table，所以也就不存在地址翻译。之后程序计数器（Program Counter）增加了4，之后下一条指令被执行时，程序计数器会被内存中的page table翻译。所以这个时刻是一个重要时刻，因为整个地址翻译从这条指令之后开始生效，指令中的每一个地址都可能对应到不同的内存地址（也就是之后的指令地址是虚拟地址）。因为在这条指令之前，我们使用的都是物理内存地址，这条指令之后page table开始生效，所有的内存地址都变成了另一个含义，也就是虚拟内存地址。

这里能正常工作的原因是值得注意的。因为前一条指令还是在物理内存地址中，后一条指令已经在虚拟内存地址中。比如，下一条指令地址是0x80001110就是一个虚拟内存地址。

![](../.gitbook/assets/image%20%28200%29.png)

为什么这里能正常工作呢？因为kernel page的映射关系中，虚拟地址到物理地址是完全相等的。所以，在我们打开虚拟地址翻译硬件之后，地址翻译硬件会将一个虚拟地址翻译到相同的物理地址。所以实际上，我们最终还是能通过内存地址执行到正确的指令，因为这里虚拟地址翻译硬件存储的物理地址就是正确的指令地址。

管理虚拟内存的一个难点是，一旦执行了类似于SATP这样的指令，你相当于将一个page table加载到了SATP寄存器，你的世界完全改变了。现在每一个地址都会被你设置好的page table所翻译。那么假设你的page table设置错误了，会发生什么呢？有人想回答这个问题吗？

> 学生A回答：你可能会覆盖kernel data。
>
> 学生B回答：会产生page fault。因为page table没有设置好，虚拟地址可能根本就翻译不了，那么内核会停止运行并panic。




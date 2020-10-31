# 4.6 kvminit 函数

接下来，让我们看一看代码，我认为很多东西都会因此变得更加清晰。

首先，我们来做一个的常规操作，启动我们的XV6，这里QEMU实现了主板，同时我们打开gdb。

![](../.gitbook/assets/image%20%28191%29.png)

上一次我们看了boot的流程，我们跟到了main函数。main函数中调用的一个函数是kvminit（3.9），这个函数会设置好kernel的地址空间。kvminit的代码如下图所示：

![](../.gitbook/assets/image%20%28199%29.png)

我们在前一部分看了kernel的地址空间长成什么样，这里我们来看一下代码是如何将它设置好的。首先在kvminit中设置一个断点，之后运行代码到断点位置。在gdb中执行layout split，可以看到（从上面的代码也可以看出）函数的第一步是为最高一级page directory分配物理page（注，调用kalloc就是分配物理page）。下一行将这段内存初始化为0。

![](../.gitbook/assets/image%20%28193%29.png)

之后，通过kvmmap函数，将每一个I/O设备映射到内核。例如，下图中高亮的行将UART0映射到内核的地址空间。

![](../.gitbook/assets/image%20%28195%29.png)

我们可以查看一个文件叫做memlayout.h，它将4.5中的文档翻译成了一堆常量。在这个文件里面可以看到，UART0对应了地址0x10000000（注，4.5中的文档是真正SiFive RISC-V的文档，而下图是QEMU的地址，所以4.5中的文档地址与这里的不符）。

![](../.gitbook/assets/image%20%28194%29.png)

所以，通过kvmmap可以将物理地址映射到相同的虚拟地址（注，因为kvmmap的前两个参数一致）。

在page table实验中，第一个练习是实现vmprint，这个函数会打印当前的kernel page table。我们现在跳过这个函数，看一下执行完第一个kvmmap时的kernel page table。

![](../.gitbook/assets/image%20%28190%29.png)

我们来看一下这里的输出。第一行是最高一级page directory的地址，这就是存在SATP或者将会存在SATP中的地址。第二行可以看到最高一级page directory只有一条PTE序号为0，它包含了中间级page directory的物理地址。第三行可以看到中间级的page directory只有一条PTE序号为128，它指向了最低级page directory的物理地址。第四行可以看到最低级的page directory包含了PTE指向物理地址。你们可以看到最低一级 page directory中PTE的物理地址就是0x10000000，对应了UART0。

前面是物理地址，我们可以从虚拟地址的角度来验证这里符合预期。我们将地址0x10000000向右移位12bit，这样可以得到虚拟地址的高27bit（index部分）。之后我们再对这部分右移位9bit，并打印成10进制数，可以得到128，这就是中间级page directory中PTE的序号。这与之前（4.4）介绍的内容是符合的。

![](../.gitbook/assets/image%20%28201%29.png)

从标志位来看（fl部分），最低一级page directory中的PTE有读写标志位，并且Valid标志位也设置了（4.3底部有标志位的介绍）。

内核会持续的按照这种方式，调用kvmmap来设置地址空间。之后会对VIRTIO0、CLINT、PLIC、kernel text、kernel data、最后是TRAMPOLINE进行地址映射。最后我们还会调用vmprint打印完整的kernel page directory，可以看出已经设置了很多PTE。

![](../.gitbook/assets/image%20%28197%29.png)

这里就不过细节了，但是这些PTE构成了我们在4.5中看到的地址空间对应关系。

（下面问答来自课程结束部分，因为内容相关就移到这里。）

> 学生：下面这两行内存不会越界吗？

![](../.gitbook/assets/image%20%28204%29.png)

> Frans：不会。这里KERNBASE是0x80000000，这是内存开始的地址。kvmmap的第三个参数是size，etext是kernel text的最后一个地址，etext - KERNBASE会返回kernel text的字节数，我不确定这块有多大，大概是60-90个page，这部分是kernel的text部分。PHYSTOP是物理内存的最大位置，PHYSTOP-text是kernel的data部分。会有足够的DRAM来完成这里的映射。


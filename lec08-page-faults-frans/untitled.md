# 8.1 Page fault Basics

今天的课程内容是page fault，以及通过page fault实现的一些列虚拟内存功能。这里相关的功能有：

* lazy allocation，这是下一个lab的内容
* copy-on-write fork
* demand paging
* memory mapped files

![](../.gitbook/assets/image%20%28307%29.png)

你懂的，几乎所有稍微正经的操作系统都实现了这些功能。如果你去看Linux，你会发现所有的这些功能都被实现了。然而在XV6，实话实说，一个这样的功能都没实现。在XV6中，一旦用户空间进程触发了page fault会导致进程被杀掉，这样的处理方式并不新鲜。

在这节课，我们将会探讨你在page fault handler中可以做的有趣的事情，进而来实现上面的功能。所以这节课对于代码的讲解会比较少，相应的会在设计层面有更多的内容，毕竟我们也没有代码可以讲解（因为没有实现）。

另一件重要的事情是，下一个实验lazy lab今天会发布出来，copy-on-write fork和mmap也是后续实验的内容。所以这些都是一个操作系统中有趣的部分，我们将会在实验中花费大量时间来研究它。

在进入到具体细节之前，先来简单回顾一下或许会有帮助。你可以认为虚拟内存有两个主要的优点：

* 第一个是Isolation，隔离性。虚拟内存使得操作系统可以给每个应用程序属于它们自己的地址空间。所以一个应用程序不可能有意或者无意的修改另一个应用程序的内存数据。虚拟内存同时也提供了用户空间和内核空间的隔离性，我们在之前的课程已经谈过很多相关内容了，你们在page table lab也可以看到。
* 另一个好处是level of indirection，提供了一层抽象。处理器和所有的指令都可以使用虚拟地址，内核会定义从虚拟地址到物理地址的映射关系。这是很多有趣功能，比如我们这节课要讨论的功能的基础。目前来说，在XV6中地址关系的映射都比较无聊，实际上在内核中基本上是直接映射（注，也就是虚拟地址等于物理地址）。当然也有几个比较有意思的地方：
  * trampoline page，它使得内核可以将一个page映射到许多用户地址空间中。
  * guard page，它同时在内核空间和用户空间用来保护Stack空间。

目前来说，这里的地址映射相对来说比较静态。不管是user还是kernel page table，我们在最开始的时候设置好，kernel就不会再对page table做任何事情了。

![](../.gitbook/assets/image%20%28222%29.png)

而page fault可以让这里的地址映射关系变得动态，通过page fault，内核可以更新内存地址映射关系，这是一个非常强大的功能。如果能结合page table和page fault，内核将会有巨大的灵活性，因为你可以动态的更新虚拟地址这一层抽象。

我们接下来会看各种各样如何利用动态重映射或者动态变更page table来实现有趣功能。



首先，我们需要思考的是，什么样的信息是必须的。当发生page fault时，内核需要什么样的信息才能够响应page fault。

* 很明显的，我们需要出错的虚拟地址，或者是触发page fault的源。可以假设的是，你们在page fault lab中已经看过一些相关的panic。内核是能够知道出错的虚拟地址的，当出现page fault的时候，内核会打印出错的虚拟地址，并且这个地址会被保存在STVAL寄存器中。所以，当一个用户应用程序触发了page fault，page fault会使用与Robert教授上节课介绍的相同的trap机制，将程序运行转到内核，同时也会将出错的地址防止在STVAL寄存器中。这是我们需要知道的第一个信息。
* 我们需要知道的第二个信息是出错的原因。我们或许想要对不同场景的page fault有不同的响应，比如因为load指令触发的page fault、因为store指令触发的page fault又或者是因为jump指令触发的page fault。所以实际上如果你查看RISC-V的文档，在SCAUSE（注，Supervisor cause寄存器，trap机制中进入到supervisor mode的原因）寄存器的介绍中，与多个与page fault相关的原因。比如，13表示是因为load引起的page fault；25表示是因为store引起的page fault；12是因为指令执行引起的page fault。所以第二个信息在SCAUSE寄存器中，总共有3中类型的原因，分别是读、写和指令。因为ECALL进入到supervisor mode对应的是8，这是我们在上节课中应该看到的SCAUSE值。基本上来说，page fault和其他的异常使用与系统调用相同的trap机制（注，详见lec07）来从用户空间转换到内核空间。如果是因为page fault触发的trap机制并且进入到内核空间，STVAL寄存器和SCAUSE寄存器都会设置好。

![](../.gitbook/assets/image%20%28259%29.png)

* 我们或许想要知道的第三个信息是触发page fault的指令的地址。从上节课可以知道，作为trap处理代码的一部分，这个地址存放在SEPC（Supervisor Exception Program Counter）寄存器，并保存在trapframe-&gt;epc中。

所以，从硬件和XV6的角度来说，当出现了page fault，我们现在有了3个对我们来说极其有价值的信息，分别是：

* 引起page fault的内存地址
* 引起page fault的原因类型
* 引起page fault时的程序计数器值，这表明了page fault在用户空间发生的位置

![](../.gitbook/assets/image%20%28296%29.png)

我们之所以关心引起page fault时的程序计数器值的原因是，在page fault handler中我们或许想要修复page table，并重新执行对应的指令。理想情况下，修复完page table之后，指令就可以无错误的运行了。所以，能够恢复因为page fault中断的指令运行是很重要的。


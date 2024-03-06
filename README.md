# 简介

因为学习MIT6.824，偶然知道了MIT6.S081这门课程。MIT6.S081这门课程的标题是Operating System Engineering，主要讲的就是操作系统。授课教授是Robert Morris和Frans Kaashoek，两位都是非常出名的程序员。

课程是基于一个类似于Unix但是简单的多的教学操作系统XV6来讲解，虽然不是原汁原味的Linux，但是对于理解Linux的工作方式和结构是足够了。与MIT6.824一样的是，这门课程是全英文，甚至英文字幕都没有。对于国内的同学来说，如果英文没有足够好，很难较好的理解这门课程。因此我计划将这门课程翻译成中文文字版。**我将在语句通顺的前提下，尽量还原课程的内容**，希望可以帮助大家学习到这门课程。如果你的英语不是那么好，建议阅读完文字再去看视频相关的课程。

目前MIT的这门课还没有上完，按计划是在12月初完成，具体的内容可以参考【1】。每一节课都在80分钟左右，大概会有6-9个知识点，我会按照独立的知识点将每节课拆分成6-9个小节。

\----------------------------------------------------------------------------------

2021-04-24 更新：

今天终于把问答课以外的20节课程都翻译完了，总共大概有35万个字，花费时间大概在200个小时左右。

这门课程相比6.824来说更像是一个整体。6.824更多的是在理解和设计分布式系统时的一些技术和技巧，而6.S081介绍了Unix风格操作系统的各个方面（虽然这两个课没什么关系(￣.￣)，但是因为是连着翻译的难免会有对比）。

实际中的操作系统会更加的复杂，但是通过这门课程的学习基本上可以对操作系统有一个全面的认识。经过翻译的过程，我自己也把之前的一些知识盲区补全了。这门课程虽然只是一个MIT的本科课程，但是我推荐给所有从事IT相关工作的同学，掌握了操作系统对于面试，debug，写代码都是有好处的。

最后，希望我的翻译可以帮助到你。

【1】[https://pdos.csail.mit.edu/6.828/2020/schedule.html](https://pdos.csail.mit.edu/6.828/2020/schedule.html)

## 如果

* 你发现了翻译的错误，可以向关联的[github](https://github.com/huihongxiao/MIT6.S081)提交PR
* 你觉得我做的还不错，可以关注我的[知乎](https://www.zhihu.com/people/xiao-hong-hui-15)，并给我一个点赞。
* 还想学习其他IT相关知识，我还做了一些其他的翻译：
  * MIT6.824 - 分布式系统：[github](https://github.com/huihongxiao/MIT6.824)；[gitbook](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824/)
  * MIT6.829 - 计算机网络 （只有前三章，已停更）：[github](https://github.com/huihongxiao/MIT6.829)；[gitbook](https://mit-public-courses-cn-translatio.gitbook.io/mit6.829/)
  * TCP拥塞控制：[github](https://github.com/huihongxiao/TCP-Congestion-Control-A-Systems-Approach)；[gitbook](https://mit-public-courses-cn-translatio.gitbook.io/tcp-congestion-control-a-systems-approach/)

## _声明_

_此次翻译纯属个人爱好，如果涉及到任何版权行为，请联系我，我将删除内容。文中所有内容，与本人现在，之前或者将来的雇佣公司无关，本人保留自省的权利，也就是说你看到的内容也不一定代表本人最新的认知和观点。_

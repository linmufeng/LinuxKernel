# LinuxKernel
《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000
原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

note : 打开链接的时候，按住ctrl在点击左键，将会在新窗口打开  

# 课程概述

本课程从理解计算机硬件的核心工作机制（存储程序计算机和函数调用堆栈）和用户态程序如何通过系统调用陷入内核（中断异常）入手，通过上下两个方向双向夹击的策略，并利用实际可运行程序的反汇编代码从实践的角度理解操作系统内核，然后开始分析Linux内核源代码，从系统调用陷入内核，进程调度与进程切换，最后返回到用户态进程，通过仔细分析梳理这一过程，并推广到硬件中断、缺页异常等内核执行路径，最终能从本质上把握Linux内核的实质，乃至在头脑中演绎Linux系统的运行过程。

[第1节 反汇编一个简单的C程序【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC1%E8%8A%82%20%E5%8F%8D%E6%B1%87%E7%BC%96%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84C%E7%A8%8B%E5%BA%8F%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第2节 一个简单的时间片轮转多道程序内核代码【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC2%E8%8A%82%20%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E6%97%B6%E9%97%B4%E7%89%87%E8%BD%AE%E8%BD%AC%E5%A4%9A%E9%81%93%E7%A8%8B%E5%BA%8F%E5%86%85%E6%A0%B8%E4%BB%A3%E7%A0%81%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第3节 跟踪分析Linux内核的启动过程【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC3%E8%8A%82%20%E8%B7%9F%E8%B8%AA%E5%88%86%E6%9E%90Linux%E5%86%85%E6%A0%B8%E7%9A%84%E5%90%AF%E5%8A%A8%E8%BF%87%E7%A8%8B%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第4节 使用库函数API和C代码中嵌入汇编代码两种方式使用同一个系统调用【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC4%E8%8A%82%20%E4%BD%BF%E7%94%A8%E5%BA%93%E5%87%BD%E6%95%B0API%E5%92%8CC%E4%BB%A3%E7%A0%81%E4%B8%AD%E5%B5%8C%E5%85%A5%E6%B1%87%E7%BC%96%E4%BB%A3%E7%A0%81%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F%E4%BD%BF%E7%94%A8%E5%90%8C%E4%B8%80%E4%B8%AA%E7%B3%BB%E7%BB%9F%E8%B0%83%E7%94%A8%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第5节 分析system_call中断处理过程【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC5%E8%8A%82%20%E5%88%86%E6%9E%90system_call%E4%B8%AD%E6%96%AD%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第6节 分析Linux内核创建一个新进程的过程【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC6%E8%8A%82%20%E5%88%86%E6%9E%90Linux%E5%86%85%E6%A0%B8%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E6%96%B0%E8%BF%9B%E7%A8%8B%E7%9A%84%E8%BF%87%E7%A8%8B%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第7节 Linux内核如何装载和启动一个可执行程序【Linux内核分析】](http://blog.csdn.net/qq470869852/article/details/69939026)

[第8节 理解进程调度时机跟踪分析进程调度与进程切换的过程【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC7%E8%8A%82%20Linux%E5%86%85%E6%A0%B8%E5%A6%82%E4%BD%95%E8%A3%85%E8%BD%BD%E5%92%8C%E5%90%AF%E5%8A%A8%E4%B8%80%E4%B8%AA%E5%8F%AF%E6%89%A7%E8%A1%8C%E7%A8%8B%E5%BA%8F%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

[第9节 Linux内核学习总结【Linux内核分析】](https://github.com/linmufeng/LinuxKernel/blob/master/%E7%AC%AC9%E8%8A%82%20Linux%E5%86%85%E6%A0%B8%E5%AD%A6%E4%B9%A0%E6%80%BB%E7%BB%93%E3%80%90Linux%E5%86%85%E6%A0%B8%E5%88%86%E6%9E%90%E3%80%91.md)

# 学习心得体会

通过半个学期的学习，要说我认为重要的不是学习到了多少内核代码（其实也很重要），毕竟仅仅是看了视频，做了几个实验，如果这就能把内核代码都搞懂，那不是学生是天才，就是老师是天才了；我觉得最重要的收获是学习Linux的方法，即从何处着手学习Linux内核，例如：如何调试内核、如何看懂内核中的汇编代码，如何分析系统调用等。这也是我学习之后最大的收获。另外就是一些有助于分析内核的工具，包括qemu、gdb等等，总之，虽然网课结束了但学习还远远没有结束。课程的实践性很强，在这里，其实老师更多的是一种启发式的学习，很多东西还是需要自己去领悟和实践

这里面我还学到了一个很有用的学习新知识的方法，就是先分析假设后寻找证据证明自己的猜想，这个过程其实是很棒的一个探究学习的体验。

课程方面，这是一门久经考验的课程，人气很高，在学完整个一个学期后，感觉学习过程很连贯，包括实验、测验、博客，设计相对来说比较合理。不过感觉后面部分，特别是后两章，视频中讲述的不是很详细，看完后需要查询很多资料才能理解和掌握操作系统操作进程的过程。比如进程切换那部分，虽然有个例子在那里，但是最好还能有更详细一些的讲解，否则，很多同学可能在那个地方对“一次调用两次返回”理解不深刻，无法体会到进程切换的奥秘。

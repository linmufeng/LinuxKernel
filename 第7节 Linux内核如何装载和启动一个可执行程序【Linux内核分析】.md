原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求
分析exec*函数对应的系统调用处理过程

# 二、实验内容
- 理解编译链接的过程和ELF可执行文件格式，详细内容参考本周第一节；
- 编程使用exec*库函数加载一个可执行文件，动态链接分为可执行程序装载时动态链接和运行时动态链接，编程练习动态链接库的这两种使用方式，详细内容参考本周第二节；
- 使用gdb跟踪分析一个execve系统调用内核处理函数sys_execve ，验证您对Linux系统加载可执行程序所需处理过程的理解，详细内容参考本周第三节；推荐在实验楼Linux虚拟机环境下完成实验。
- 特别关注新的可执行程序是从哪里开始执行的？为什么execve系统调用返回后新的可执行程序能顺利执行？对于静态链接的可执行程序和动态链接的可执行程序execve系统调用返回时会有什么不同？

# 三、实验环境

本地linux环境（ubuntu14.04 64bit）

主要优点：使用方便，方便保存，不受网络影响。

# 四、实验过程

## 1. 跟踪分析一个execve系统调用内核处理函数sys_execve。

将test.c替换为test_exec.c，仍命名为test.c，为menu增加exec功能。

```
cd LinuxKernel
cd menu
mv test_exec.c test.cd
```

新的test.c的main函数中为界面增加了exec的选项。

![这里写图片描述](http://img.blog.csdn.net/20170409231235425?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

查看exec本身代码内容。就是一个简单的程序。子进程执行了hello。

![这里写图片描述](http://img.blog.csdn.net/20170409231303899?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

重新编译运行menu，然后输入exec命令，来看一下最终执行的效果。

 重新加上-s –S运行，然后利用gdb跟踪，设3个断点：sys\_execve、load_elf_binary、start_thread。


  继续运行，启动menu过程中也会触发断点，不要理会，继续就好。待menu启动完成，输入exec命令，这时会停在第一个断点。

可以看到接下来真正运行的是SYSCALL_DEFINE3，查看这部分代码。



# 五、代码分析

## 1.

# 六、总结
新的可执行程序是从new\_ip开始执行,start_thread实际上是把返回到用户态的位置从Int 0x80的下一条指令，变成了规定的新加载的可执行文件的入口位置,即修改内核堆栈的EIP的值作为新程序的起点。
当执行到execve系统调用时，陷入内核态，用execve加载的可执行文件覆盖当前进程的可执行程序，当execve系统调用返回时，返回新的可执行程序的执行起点（main函数位置），所以execve系统调用返回后新的可执行程序能顺利执行。
对于静态链接的可执行程序和动态链接的可执行程序execve系统调用返回时，如果是静态链接，elf\_entry指向可执行文件规定的头部（main函数对应的位置0x8048\***）；如果需要依赖动态链接库，elf_entry指向动态链接器的起点。动态链接主要是由动态链接器ld来完成的。

# 参考资料

原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

### 一、实验要求
分析exec*函数对应的系统调用处理过程

### 二、实验内容
- 理解编译链接的过程和ELF可执行文件格式，详细内容参考本周第一节；
- 编程使用exec*库函数加载一个可执行文件，动态链接分为可执行程序装载时动态链接和运行时动态链接，编程练习动态链接库的这两种使用方式，详细内容参考本周第二节；
- 使用gdb跟踪分析一个execve系统调用内核处理函数sys_execve ，验证您对Linux系统加载可执行程序所需处理过程的理解，详细内容参考本周第三节；推荐在实验楼Linux虚拟机环境下完成实验。
- 特别关注新的可执行程序是从哪里开始执行的？为什么execve系统调用返回后新的可执行程序能顺利执行？对于静态链接的可执行程序和动态链接的可执行程序execve系统调用返回时会有什么不同？

### 三、实验环境

本地linux环境（ubuntu14.04 64bit）

主要优点：使用方便，方便保存，不受网络影响。

## 四、实验过程

### 1.可执行文件的生成过程。

首先引用一张孟老师的图，来说明可执行文件生成的过程，总结的很赞

![这里写图片描述](http://img.blog.csdn.net/20170411200314118?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

可执行文件是给计算机中的cpu执行的二进制代码。按照老师上课使用的shell命令演示一下。

![这里写图片描述](http://img.blog.csdn.net/20170411201006332?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

hello.static 是静态链接编译的可执行文件，可以很看到，比动态链接编译的hello文件要大得多，相差100倍，原因也很简单，就是静态链接是将程序中需要使用的库都放在了静态编译的文件中，导致文件大小剧增。

演示很简单，两个的效果表面是看不出来的

![这里写图片描述](http://img.blog.csdn.net/20170411201352478?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 2.查看elf文件相关信息

对于静态链接的elf文件，基本上在加载时对应加上程序入口地址，将相应的代码数据加载到对应的内存空间中，然后逐步执行代码。以下是我的ELF Header情况。

![这里写图片描述](http://img.blog.csdn.net/20170411201833784?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面来查看elf头信息,从图中可以看出真正的代码从0x8048320开始，这是程序载入内存被执行的真正入口

![这里写图片描述](http://img.blog.csdn.net/20170411201616627?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


### 3.动态链接库
通常我们的程序还需要使用动态链接库。分为装载时动态链接和运行时动态链接。

[孟老师提供的演示代码](http://mooc.study.163.com/course/attachment.htm?fileName=SharedLibDynamicLink.zip&nosKey=DF0B57B0514B0357EA08D0DCA4538B4A-1427446685857)

生成共享库和运行时链接库：
```
gcc -shared shlibexample.c -o libshlibexample.so -m32
gcc -shared dllibexample.c -o libdllibexample.so -m32
```

生成的库文件如下，并运行

![这里写图片描述](http://img.blog.csdn.net/20170411202303896?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 4. 跟踪execlp的调用

这次跟踪分析一个execve系统调用内核处理函数sys_execve。

#### 1.将test.c替换为test_exec.c，为MenuOS增加exec功能。

```
cd LinuxKernel
cd menu
mv test_exec.c test.c
```

新的test.c的main函数中为界面增加了exec的选项。

![这里写图片描述](http://img.blog.csdn.net/20170411203141509?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

查看exec本身代码内容。就是一个简单的程序。子进程执行了hello。

![这里写图片描述](http://img.blog.csdn.net/20170411203204087?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

#### 2.调试MenuOS

编译运行menu，因为前面几个实验都做了无数遍了，就不多浪费篇幅。加上-s –S运行，然后利用gdb跟踪，设3个断点：sys\_execve、load\_elf_binary、start_thread。

![这里写图片描述](http://img.blog.csdn.net/20170411203729259?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

continue继续运行，启动menu过程中会触发断点，直接继续执行。menu启动完成后，输入exec命令，这时会停在第一个断点。

![这里写图片描述](http://img.blog.csdn.net/20170411204137046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

 继续运行，停在第二个断点，是装载的过程。

![这里写图片描述](http://img.blog.csdn.net/20170411204738507?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170411204631611?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

继续运行，停在第三个断点。

![这里写图片描述](http://img.blog.csdn.net/20170411205025879?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

这是注意到有一个新的ip地址new_ip作为参数传递过来。打印后发现这个地址是0x8048d2a，其内容目前无法访问。

这时另开一个终端窗口，打印hello的头文件，发现就是hello可执行程序的起始位置，即用户态第一条指令的位置。

![这里写图片描述](http://img.blog.csdn.net/20170411205128668?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行到start_thread代码。
 
![这里写图片描述](http://img.blog.csdn.net/20170411205509538?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下面是start_kernel的源代码

```
start_thread(struct pt_regs *regs, unsigned long new_ip, unsigned long new_sp)
{
    set_user_gs(regs, 0);
    regs->fs        = 0;
    regs->ds        = __USER_DS;
    regs->es        = __USER_DS;
    regs->ss        = __USER_DS;
    regs->cs        = __USER_CS;
    regs->ip        = new_ip;
    regs->sp        = new_sp;
    regs->flags        = X86_EFLAGS_IF;
    /*
     * force it to the iret return path by making it look as if there was
     * some work pending.
     */
    set_thread_flag(TIF_NOTIFY_RESUME);
}
```

单步运行。可以看到对寄存器的修改情况。

![这里写图片描述](http://img.blog.csdn.net/20170411205606992?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

execve在返回前用新的ip和sp更新了进程的ip和sp。对于需要动态链接的程序，elf_entry就会加载动态链接器ld的入口地址。

继续执行，程序就将进入用户态执行

![这里写图片描述](http://img.blog.csdn.net/20170411205812071?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



## 五、代码分析

execve和前面博文分析的fork系统一样，是一种特殊的系统调用。fork的特殊在于系统调用后两次返回，生成了新进程，而不单单是在原来程序的系统调用的下一条语句。而execve的特殊在于它返回之后，执行的是一个新的程序了（例如返回程序的main入口，修改的是elf_entry），而不是以前调用execve的进程shell了。
内核处理函数sys_execve内部会解析可执行文件格式，它的内部执行流程是do_execve -> do_execve_common -> exec_binprm。
gdb断点设置：b sys_execve ;停到该位置，继续设置断点 b load_elf_binary; b start_thread。
其中的一些函数解释：
1）search_binary_handler符合寻找文件格式对应的解析模块，如下：

```
list_for_each_entry(fmt, &formats, lh) {
if (!try_module_get(fmt->module))
continue;
read_unlock(&binfmt_lock);
bprm->recursion_depth++;
retval = fmt->load_binary(bprm);
read_lock(&binfmt_lock);
```

对于ELF格式的可执行文件fmt->load_binary(bprm);执行的应该是load_elf_binary

2）Linux内核是如何支持多种不同的可执行文件格式的？

```
static struct linux_binfmt elf_format = {
.module = THIS_MODULE,
.load_binary = load_elf_binary,
.load_shlib = load_elf_library,
.core_dump = elf_core_dump,
.min_coredump = ELF_EXEC_PAGESIZE,
};
static int __init init_elf_binfmt(void)
{
register_binfmt(&elf_format);
return 0;
}
```

elf\_format 和 init_elf_binfmt，就是观察者模式中的观察者。

3)可执行文件开始执行的起点在哪里？如何才能让execve系统调用返回到用户态时执行新程序？
load_elf_binary -> start_thread中通过修改内核堆栈中的EIP的值作为新程序的起点。即修改一开始int 0x80压入内核堆栈的EIP。start_thread中的new_ip是返回到用户态第一条指令的地址，与可执行程序的头中的入口地址相同。



# 六、总结
新的可执行程序是从new\_ip开始执行,start_thread实际上是把返回到用户态的位置从Int 0x80的下一条指令，变成了规定的新加载的可执行文件的入口位置,即修改内核堆栈的EIP的值作为新程序的起点。
当执行到execve系统调用时，陷入内核态，用execve加载的可执行文件覆盖当前进程的可执行程序，当execve系统调用返回时，返回新的可执行程序的执行起点（main函数位置），所以execve系统调用返回后新的可执行程序能顺利执行。
对于静态链接的可执行程序和动态链接的可执行程序execve系统调用返回时，如果是静态链接，elf\_entry指向可执行文件规定的头部（main函数对应的位置0x8048\***）；如果需要依赖动态链接库，elf_entry指向动态链接器的起点。动态链接主要是由动态链接器ld来完成的。

# 参考资料

[1. 静态库、共享库和动态加载库](http://blog.csdn.net/since20140504/article/details/45057619)

[2. linux程序的命令行参数](http://blog.csdn.net/dog250/article/details/5303629)

[3. Linux内核如何装载和启动一个可执行程序](http://swordautumn.blog.51cto.com/1485402/1633663/)

[4. Linux内核如何装载和启动一个可执行程序](http://m.blog.csdn.net/article/details?id=51111604)

[5. Linux内核如何装载和启动一个可执行程序](https://xuezhaojiang.github.io/LinuxCore/lab7/lab7.html)

[6. Linux内核如何装载和启动一个可执行程序]()

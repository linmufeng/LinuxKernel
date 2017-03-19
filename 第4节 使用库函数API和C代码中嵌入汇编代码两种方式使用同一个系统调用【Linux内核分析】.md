原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求
- 选择一个系统调用（13号系统调用time除外），系统调用列表参见[Cross Reference: syscall_32.tbl](http://codelab.shiyanlou.com/xref/linux-3.18.6/arch/x86/syscalls/syscall_32.tbl)

- 参考视频中的方式使用库函数API和C代码中嵌入汇编代码两种方式使用同一个系统调用

# 二、基础知识
1. 系统调用
由操作系统实现提供的所有系统调用所构成的集合即程序接口或应用编程接口(Application Programming Interface，API)。是应用程序同系统之间的接口。
操作系统的主要功能是为管理硬件资源和为应用程序开发人员提供良好的环境来使应用程序具有更好的兼容性，为了达到这个目的，内核提供一系列具备预定功能的多内核函数，通过一组称为系统调用（system call)的接口呈现给用户。系统调用把应用程序的请求传给内核，调用相应的的内核函数完成所需的处理，将处理结果返回给应用程序。

2. 系统调用的三层皮
> ![这里写图片描述](http://img.blog.csdn.net/20170319195155763?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

- API：第一层是指Libc中定义的API，这些API封装了系统调用，使用int 0x80触发一个系统调用中断；当然，并非所有的API都使用了系统调用，如完成数学加减运算的API就没有使用系统调用；也有可能某个API使用了多个系统调用；这一层存在的价值就是为应用程序员提供易于使用的API来调用系统调用；

- system\_call：运行于内核态。system_call是所有系统调用在内核的入口点，在其中的开始处保护用户态程序执行上下文，结束处恢复用户态程序执行上下文，在中间根据传入的系统调用号对应的中断服务程序；

- sys\_xyz 系统调用封装例程：执行具体的系统调用操作，完成用户的系统调用请求；每个系统调用都对应一个封装例程；

# 三、实验环境

实验楼linux内核分析课程[线上虚拟机的linux环境](http://www.shiyanlou.com/courses/195)

主要优点：环境免配置，使用方便，不消耗主机资源。

# 四、实验过程

## 1.实验说明
由基础知识部分可以知道，请求一个系统调用，一般有两种方法可以实现，一是使用Libc提供的API，二是直接在C中内嵌汇编代码触发0x80中断来完成。

## 2.使用getpid 系统调用

### 使用库函数API获取pid

![这里写图片描述](http://img.blog.csdn.net/20170319193552787?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 使用C语言内嵌汇编带码实现

![这里写图片描述](http://img.blog.csdn.net/20170319193543599?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

### 运行结果
![这里写图片描述](http://img.blog.csdn.net/20170319194753964?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 五、代码分析


```
#include <stdio.h>
#include <time.h>
int main()
{
    unsigned pid;
    asm volatile(
        "mov $0,%%ebx\n\t"
        "mov $20,%%eax\n\t"
        "int $0x80\n\t"
        "mov %%eax,%0\n\t"
        :"=m"(pid)
    );
    printf("pid is %u\nget pid by asm\n",pid);
    return 0;
}
```
在系统调用列表[Cross Reference: syscall_32.tbl](http://codelab.shiyanlou.com/xref/linux-3.18.6/arch/x86/syscalls/syscall_32.tbl) 中选择了20号系统调用getpid，该系统调用的作用是获取当前进程的进程号。系统调用采用eax传递系统调用号。getpid的系统调用号是20，也就是要将调用号20存入寄存器eax。然后执行系统调用是通过执行int $0x80。



# 六、总结
系统调用是用户态与内核态的桥梁，而具体的措施就是中断。通过本实验，更加熟悉了系统调用的本质，以及系统调用和中断的关联。应用程序在用户态调用API函数，该函数将对应的系统调用号及参数保存，触发软中断，然后陷入内核态，system\_call根据系统调用号调用对应的内核函数，内核函数执行完毕后将结果存放的eax中并返回给程序，程序返回的用户态。

# 参考资料

 [系统调用](http://baike.baidu.com/link?url=zcKqTAX6EAl7u8bu4YjWms2MrGF2-4RiJ8jvqfiKucTd3O7UM8-pPxJcPTF3zbTbnhqsxFC4k7XwP9XKLjbMFXrDT32pAQvh7PQq82C-waKCaIw3YrOvtPwbcmxV_IFN)
- [Linux内核分析](http://www.tuicool.com/articles/E7rYza)

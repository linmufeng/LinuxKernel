原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求

完成一个简单的时间片轮转多道程序内核代码。

# 二、实验内容

- 完成一个简单的时间片轮转多道程序内核代码，代码见视频中或从mykernel找
- 分析进程的启动和进程的切换机制
- 理解操作系统如何工作

# 三、实验环境

实验楼[linux内核分析](https://www.shiyanlou.com/courses/195)课程线上虚拟机的linux环境

主要优点：环境免配置，使用方便，不消耗主机资源。

# 四、实验过程

## 1.代码获取
代码来自于：[孟宁老师的github版本库](https://github.com/mengning/mykernel)

共拷贝了三个文件，将其放置在实验环境中的LinuxKernel/linux-3.9.4/mykernel中。

> mypcb.h
> mymain.c 
> myinterrupt.c

## 2. 编译内核代码并执行

编译内核代码
```
make
```

![这里写图片描述](http://img.blog.csdn.net/20170305184827347?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行
```
qemu -kernel arch/x86/boot/bzImage
```

![这里写图片描述](http://img.blog.csdn.net/20170303203018447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> 运行效果截图

![这里写图片描述](http://img.blog.csdn.net/20170305184958533?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 五、linux内核分析

## 1.代码分析

### mypcb.h

```
/*
 *  linux/mykernel/mypcb.h
 *
 *  Kernel internal PCB types
 *
 *  Copyright (C) 2013  Mengning
 *
 */
#define MAX_TASK_NUM        4
#define KERNEL_STACK_SIZE   1024*8
/* CPU-specific state of this task */
struct Thread {
    unsigned long		ip;
    unsigned long		sp;
};
typedef struct PCB{
    int pid;
    volatile long state;	/* -1 unrunnable, 0 runnable, >0 stopped */
    char stack[KERNEL_STACK_SIZE];
    /* CPU-specific state of this task */
    struct Thread thread;
    unsigned long	task_entry;
    struct PCB *next;
}tPCB;
void my_schedule(void);
```

mypcb.h定义了

- 最大进程数 MAX_TASK_NUM 
即有MAX_TASK_NUM个进程参与内核的时间片轮转调度。
- 内核进程栈的大小 KERNEL_STACK_SIZE
即每一个进程可以使用的堆栈大小。
- 线程Thread
包括指令指针ip和栈顶指针sp。
- 进程控制块PCB
包括进程编号、进程状态state、进程堆栈、线程、任务实体、下一个进程指针
- 调度函数my_schedule
用于模拟进程执行一段时间后切换到其他进程继续执行的过程


### mymain.c

```
/*
 *  linux/mykernel/mymain.c
 *
 *  Kernel internal my_start_kernel
 *
 *  Copyright (C) 2013  Mengning
 *
 */
#include <linux/types.h>
#include <linux/string.h>
#include <linux/ctype.h>
#include <linux/tty.h>
#include <linux/vmalloc.h>


#include "mypcb.h"

tPCB task[MAX_TASK_NUM];
tPCB * my_current_task = NULL;
volatile int my_need_sched = 0;

void my_process(void);


void __init my_start_kernel(void)
{
    int pid = 0;
    int i;
    /* Initialize process 0*/
    task[pid].pid = pid;
    task[pid].state = 0;/* -1 unrunnable, 0 runnable, >0 stopped */
    task[pid].task_entry = task[pid].thread.ip = (unsigned long)my_process;
    task[pid].thread.sp = (unsigned long)&task[pid].stack[KERNEL_STACK_SIZE-1];
    task[pid].next = &task[pid];
    /*fork more process */
    for(i=1;i<MAX_TASK_NUM;i++)
    {
        memcpy(&task[i],&task[0],sizeof(tPCB));
        task[i].pid = i;
        task[i].thread.sp = (unsigned long)&task[i].stack[KERNEL_STACK_SIZE-1];
	*(task[i].thread.sp - 1) = task[i].thread.sp;
	task[i].thread.sp -= 1;
        task[i].next = task[i-1].next;
        task[i-1].next = &task[i];
    }
    /* start process 0 by task[0] */
    pid = 0;
    my_current_task = &task[pid];
	asm volatile(
    	"movl %1,%%esp\n\t" 	/* set task[pid].thread.sp to esp */
    	"pushl %1\n\t" 	        /* push ebp */
    	"pushl %0\n\t" 	        /* push task[pid].thread.ip */
    	"ret\n\t" 	            /* pop task[pid].thread.ip to eip */
    	"popl %%ebp\n\t"
    	: 
    	: "c" (task[pid].thread.ip),"d" (task[pid].thread.sp)	/* input c or d mean %ecx/%edx*/
	);
}   
void my_process(void)
{
    int i = 0;
    while(1)
    {
        i++;
        if(i%10000000 == 0)
        {
            printk(KERN_NOTICE "this is process %d -\n",my_current_task->pid);
            if(my_need_sched == 1)
            {
                my_need_sched = 0;
        	    my_schedule();
        	}
        	printk(KERN_NOTICE "this is process %d +\n",my_current_task->pid);
        }     
    }
}
```
mymain.c中，主要是进程的初始化并负责启动进程，做了三件事

1. 初始化一个进程
为其分配进程编号、进程状态state、进程堆栈、线程、任务实体等，并将其next指针指向自己。
2. 初始化更多的进程
根据第一个进程的部分资源，包括内存拷贝函数的运用，将0号进程的信息进行了复制，修改pid等信息。
3. 设置当前进程
因为是初始化，所以当前进程就决定给0号进程了，通过执行嵌入式汇编代码，开始执行mykernel内核。

重点分析一下嵌入式汇编代码：
```
asm volatile(
        "movl %1,%%esp\n\t"     /* set task[pid].thread.sp to esp */
        "pushl %1\n\t"          /* push ebp */
        "pushl %0\n\t"          /* push task[pid].thread.ip */
        "ret\n\t"               /* pop task[pid].thread.ip to eip */
        "popl %%ebp\n\t"
        : 
        : "c" (task[pid].thread.ip),"d" (task[pid].thread.sp)   /* input c or d mean %ecx/%edx*/
    );
```

首先解释一下含有冒号的部分，因为这一部分我之前不太明白。"c"就是ecx，内容为task[pid].thread.ip，"d"就是edx，内容为task[pid].thread.sp。从冒号开始，以逗号分隔开，从0开始，每一个寄存器有一个标号，即ecx的标号为%0，edx的标号为%1。

接着对上面的汇编代码进行分析，从第二行开始，第二行将esp置为标号%1，即将esp指向当前进程的栈顶；第三行将标号%1压栈，因为上一句就是当前进程的esp被压栈，此时栈为空，当前ebp被压栈，esp下移一位；第四行将标号%0压栈，即当前进程的指令指针eip压栈，第五行将eip指向当前进程指针，开始执行当前进程。

然后实现了my_process函数，这是一个死循环，每个进程都是执行此函数。每10000000次，打印当前进程的pid，全局变量my_need_sched，通过对my_need_sched进行判断，若为1，则通知正在执行的进程执行调度程序，然后打印调度后的进程pid。


### myinterrupt.c
```
/*
 *  linux/mykernel/myinterrupt.c
 *
 *  Kernel internal my_timer_handler
 *
 *  Copyright (C) 2013  Mengning
 *
 */
#include <linux/types.h>
#include <linux/string.h>
#include <linux/ctype.h>
#include <linux/tty.h>
#include <linux/vmalloc.h>

#include "mypcb.h"

extern tPCB task[MAX_TASK_NUM];
extern tPCB * my_current_task;
extern volatile int my_need_sched;
volatile int time_count = 0;

/*
 * Called by timer interrupt.
 * it runs in the name of current running process,
 * so it use kernel stack of current running process
 */
void my_timer_handler(void)
{
#if 1
    if(time_count%1000 == 0 && my_need_sched != 1)
    {
        printk(KERN_NOTICE ">>>my_timer_handler here<<<\n");
        my_need_sched = 1;
    } 
    time_count ++ ;  
#endif
    return;  	
}

void my_schedule(void)
{
    tPCB * next;
    tPCB * prev;

    if(my_current_task == NULL 
        || my_current_task->next == NULL)
    {
    	return;
    }
    printk(KERN_NOTICE ">>>my_schedule<<<\n");
    /* schedule */
    next = my_current_task->next;
    prev = my_current_task;
    if(next->state == 0)/* -1 unrunnable, 0 runnable, >0 stopped */
    {
    	my_current_task = next; 
    	printk(KERN_NOTICE ">>>switch %d to %d<<<\n",prev->pid,next->pid);  
    	/* switch to next process */
    	asm volatile(	
        	"pushl %%ebp\n\t" 	    /* save ebp */
        	"movl %%esp,%0\n\t" 	/* save esp */
        	"movl %2,%%esp\n\t"     /* restore  esp */
        	"movl $1f,%1\n\t"       /* save eip */	
        	"pushl %3\n\t" 
        	"ret\n\t" 	            /* restore  eip */
        	"1:\t"                  /* next process start here */
        	"popl %%ebp\n\t"
        	: "=m" (prev->thread.sp),"=m" (prev->thread.ip)
        	: "m" (next->thread.sp),"m" (next->thread.ip)
    	); 
 	
    }
    else
    {
        next->state = 0;
        my_current_task = next;
        printk(KERN_NOTICE ">>>switch %d to %d<<<\n",prev->pid,next->pid);
    	/* switch to new process */
    	asm volatile(	
        	"pushl %%ebp\n\t" 	    /* save ebp */
        	"movl %%esp,%0\n\t" 	/* save esp */
        	"movl %2,%%esp\n\t"     /* restore  esp */
        	"movl $1f,%1\n\t"       /* save eip */	
        	"pushl %3\n\t" 
        	"ret\n\t" 	            /* restore  eip */
        	"movl %2,%%ebp\n\t"     /* restore  ebp */
        	: "=m" (prev->thread.sp),"=m" (prev->thread.ip)
        	: "m" (next->thread.sp),"m" (next->thread.ip)
    	);          
    }   
    return;	
}
``` 

### 定时中断函数
实现比较简单，就是实现定时器中断，每1000下进行my_need_sched的检查，如果不为1，则置其为1使其进程调度。

my_schedule函数具体实现了进程的切换。声明了两个指针，prev和next，分别指向当前进程和下一个进程。进程切换时分两种情况，当next_state ==0 时，即下一个进程正在执行。

在my_schedule函数中，完成进程的切换。进程的切换分两种情况，一种情况是下一个进程没有被调度过，另外一种情况是下一个进程被调度过，可以通过下一个进程的state知道其状态。进程切换依然是通过内联汇编代码实现，无非是保存旧进程的eip和堆栈，将新进程的eip和堆栈的值存入对应的寄存器中。

```
    	/* switch to next process */
    	asm volatile(	
        	"pushl %%ebp\n\t" 	    /* save ebp */
        	"movl %%esp,%0\n\t" 	/* save esp */
        	"movl %2,%%esp\n\t"     /* restore  esp */
        	"movl $1f,%1\n\t"       /* save eip */	
        	"pushl %3\n\t" 
        	"ret\n\t" 	            /* restore  eip */
        	"1:\t"                  /* next process start here */
        	"popl %%ebp\n\t"
        	: "=m" (prev->thread.sp),"=m" (prev->thread.ip)
        	: "m" (next->thread.sp),"m" (next->thread.ip)
```
上边代码实现了进程的切换，第三行将当前的ebp保存压栈，然后将当前的esp存入内存中（prev-&gt;thread.sp）；第四行将ebp指向要切换进程的esp；第五行保存当前的eip到pre-&gt;thread.ip；第六行将下一个进程的eip压栈；第七行ret实现将栈顶，即刚刚压栈的eip弹出，赋给eip，程序开始从此处执行（即要切换的进程），完成了进程切换。

当下一个进程状态不为0时，即表示还未执行，此时esp等与ebp，其余部分和第一种情况相同。

# 六、实验知识点回顾

本次实验首先总结了一下linux操作系统的工作方式，并且了解到了计算机工作的三大法宝，而且了解了堆栈的工作方式和如何在C语言中嵌套汇编语言的方法。

懂得了如何在Linux系统中查看相应的代码，然后进行了一段时间片轮转的操作系统内核代码的分析，跟着学习视频一步步的进行学习，虽然这样的分析挺难的，但是还是成功的完成了。

# 七、实验总结

通过本讲的学习和实验，我们知道操作系统的核心功能就是：进程调度和中断机制，通过与硬件的配合实现多任务处理，再加上上层应用软件的支持，最终变成可以使用户可以很容易操作的计算机系统。  

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

![这里写图片描述](http://img.blog.csdn.net/20170303202819704?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

执行
```
qemu -kernel arch/x86/boot/bzImage
```

![这里写图片描述](http://img.blog.csdn.net/20170303203018447?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

> 运行效果截图

![这里写图片描述](http://img.blog.csdn.net/20170303203223104?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

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
    return;	
}
``` 




# 六、实验知识点回顾

# 七、实验总结

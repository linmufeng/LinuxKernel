原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求
根据本周所学知识分析fork函数对应的系统调用处理过程

# 二、实验内容
- 阅读理解task_struct数据结构http://codelab.shiyanlou.com/xref/linux-3.18.6/include/linux/sched.h#1235；

- 分析fork函数对应的内核处理过程sys\_clone，理解创建一个新进程如何创建和修改task\_struct 的数据结构；

- 使用gdb跟踪分析一个fork系统调用内核处理函数sys\_clone ，验证您对Linux系统创建一个新进程的理解,推荐在实验楼Linux虚拟机环境下完成实验。

- 特别关注新进程是从哪里开始执行的？为什么从哪里能顺利执行下去？即执行起点与内核堆栈如何保证一致。

# 三、实验环境

本地linux环境（ubuntu14.04 64bit）

主要优点：使用方便，方便保存，不受网络影响。

# 四、实验过程

## 1. 在本地搭建环境
以前都是在虚拟机上完成的实验,这次就在本地搭建了实验环境,大部分过程参考[孟老师的教程](https://github.com/mengning/mykernel/blob/master/README.md)。

为了方便查看，特在此将完整过程写下，使用自己的Linux系统环境搭建MenuOS的过程：

```
    # 下载内核源代码编译内核
    cd ~/LinuxKernel/
    wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.18.6.tar.xz
    xz -d linux-3.18.6.tar.xz
    tar -xvf linux-3.18.6.tar
    cd linux-3.18.6
    make i386_defconfig
    make # 一般要编译很长时间，少则20分钟多则数小时
     
    # 制作根文件系统
    cd ~/LinuxKernel/
    mkdir rootfs
    git clone https://github.com/mengning/menu.git  # 如果被墙，可以使用附件menu.zip 
    cd menu
    gcc -o init linktable.c menu.c test_fork.c -m32 -static –lpthread
    cd ../rootfs
    cp ../menu/init ./
    find . | cpio -o -Hnewc |gzip -9 > ../rootfs.img
     
    # 启动MenuOS系统
    cd ~/LinuxKernel/
    qemu -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img
```

注意：
制作根文件系统，如果你的系统是新安装的，那么很可能就无法制作成功，会出现错误
```
/usr/include/features.h:374:25: 
fatal error: sys/cdefs.h: No such file or directory
```
原因很简单，就是因为缺少c/c++的函数库。运行下面两句代码，完成库的安装
```
sudo apt-get install gcc-multilib
sudo apt-get install g++-multilib
```

> 解决方法参考自：[64bit Linux tries to compile 32bit and fails](https://github.com/couchbase/couchbase-lite-java-native/issues/11)

## 2.启动MenuOS

打开shell终端，执行以下命令：

```
cd LinuxKernel

rm -rf menu

git clone https://github.com/mengning/menu.git

cd menu

mv test_fork.c test.c

make rootfs
```

![这里写图片描述](http://img.blog.csdn.net/20170403141500899?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 3.调试MenuOS

通过增加-s -S启动参数打开调试模式

```
qemu -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img -s -S
```

打开gdb进行远程调试

```
gdb

file linux-3.18.6/vmlinux

target remote:1234
```

![这里写图片描述](http://img.blog.csdn.net/20170403141601078?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

设置断点

```
b sys_clone

b do_fork

b dup_task_struct

b copy_process

b copy_thread

b ret_from_fork
```

![这里写图片描述](http://img.blog.csdn.net/20170403141637422?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


在系统启动过程走了些弯路，设置断点的这几个函数都是系统创建进程所必须的关键部分，所以启动过程中do\_fork,copy_process，copy_thread不断的多次出现，我只好暂时使断点失效，才让menuos顺利启动到命令提示符。disable breakpoints 1 2 3 4 5 6.
执行fork命令，停在了断点SyS\_clone处，单步执行，定在了断点do_fork处。经过几行代码，

![这里写图片描述](http://img.blog.csdn.net/20170403141826860?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

p = copy\_process(clone_flags, stack_start,stack_size,chile_tidptr, NULL,trace);
停在断点copy_process.在执行几行代码，如下：
p = dup\_task_struct(current);
继续执行，断点arch_dup_task_struct停住。
连续n命令后可以看到子进程的初始化过程：

 ![这里写图片描述](http://img.blog.csdn.net/20170403142141392?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
 
程序执行断在copy_thread后如下：

![这里写图片描述](http://img.blog.csdn.net/20170403142215410?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


执行finish,及continue命令，进入了子进程执行的起点ret_from_fork.
 
![这里写图片描述](http://img.blog.csdn.net/20170403142300737?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast) 

执行c命令，主要过程基本结束。


# 五、代码分析

## 1.进程描述

进程描述符（task_struct）

用来描述进程的数据结构，可以理解为进程的属性。比如进程的状态、进程的标识（PID）等，都被封装在了进程描述符这个数据结构中，该数据结构被定义为task_struct

进程控制块（PCB）

是操作系统核心中一种数据结构，主要表示进程状态。

## 2.fork一个子进程的代码

```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
int main(int argc, char * argv[])
{
    int pid;
    /* fork another process */
    pid = fork();
    if (pid < 0) 
    { 
        /* error occurred */
        fprintf(stderr,"Fork Failed!");
        exit(-1);
    } 
    else if (pid == 0) 
    {
        /* child process */
        printf("This is Child Process!\n");
    } 
    else 
    { 
        /* parent process  */
        printf("This is Parent Process!\n");
        /* parent will wait for the child to complete*/
        wait(NULL);
        printf("Child Complete!\n");
    }
}
```

## 3.进程创建

### 大致流程

fork 通过0x80中断（系统调用）来陷入内核，由系统提供的相应系统调用来完成进程的创建。

fork.c

```
//fork
#ifdef __ARCH_WANT_SYS_FORK
SYSCALL_DEFINE0(fork)
{
#ifdef CONFIG_MMU
    return do_fork(SIGCHLD, 0, 0, NULL, NULL);
#else
    /* can not support in nommu mode */
    return -EINVAL;
#endif
}
#endif

//vfork
#ifdef __ARCH_WANT_SYS_VFORK
SYSCALL_DEFINE0(vfork)
{
    return do_fork(CLONE_VFORK | CLONE_VM | SIGCHLD, 0,
            0, NULL, NULL);
}
#endif

//clone
#ifdef __ARCH_WANT_SYS_CLONE
#ifdef CONFIG_CLONE_BACKWARDS
SYSCALL_DEFINE5(clone, unsigned long, clone_flags, unsigned long, newsp,
         int __user *, parent_tidptr,
         int, tls_val,
         int __user *, child_tidptr)
#elif defined(CONFIG_CLONE_BACKWARDS2)
SYSCALL_DEFINE5(clone, unsigned long, newsp, unsigned long, clone_flags,
         int __user *, parent_tidptr,
         int __user *, child_tidptr,
         int, tls_val)
#elif defined(CONFIG_CLONE_BACKWARDS3)
SYSCALL_DEFINE6(clone, unsigned long, clone_flags, unsigned long, newsp,
        int, stack_size,
        int __user *, parent_tidptr,
        int __user *, child_tidptr,
        int, tls_val)
#else
SYSCALL_DEFINE5(clone, unsigned long, clone_flags, unsigned long, newsp,
         int __user *, parent_tidptr,
         int __user *, child_tidptr,
         int, tls_val)
#endif
{
    return do_fork(clone_flags, newsp, 0, parent_tidptr, child_tidptr);
}
#endif
```

通过看上边的代码，我们可以清楚的看到，不论是使用 fork 还是 vfork 来创建进程，最终都是通过 do\_fork() 方法来实现的。接下来我们可以追踪到 do\_fork()的代码（部分代码，经过精简）：

```
long do_fork(unsigned long clone_flags,
          unsigned long stack_start,
          unsigned long stack_size,
          int __user *parent_tidptr,
          int __user *child_tidptr)
{
        //创建进程描述符指针
        struct task_struct *p;

        //……

        //复制进程描述符，copy_process()的返回值是一个 task_struct 指针。
        p = copy_process(clone_flags, stack_start, stack_size,
             child_tidptr, NULL, trace);

        if (!IS_ERR(p)) {
            struct completion vfork;
            struct pid *pid;

            trace_sched_process_fork(current, p);

            //得到新创建的进程描述符中的pid
            pid = get_task_pid(p, PIDTYPE_PID);
            nr = pid_vnr(pid);

            if (clone_flags & CLONE_PARENT_SETTID)
                put_user(nr, parent_tidptr);

            //如果调用的 vfork()方法，初始化 vfork 完成处理信息。
            if (clone_flags & CLONE_VFORK) {
                p->vfork_done = &vfork;
                init_completion(&vfork);
                get_task_struct(p);
            }

            //将子进程加入到调度器中，为其分配 CPU，准备执行
            wake_up_new_task(p);

            //fork 完成，子进程即将开始运行
            if (unlikely(trace))
                ptrace_event_pid(trace, pid);

            //如果是 vfork，将父进程加入至等待队列，等待子进程完成
            if (clone_flags & CLONE_VFORK) {
                if (!wait_for_vfork_done(p, &vfork))
                    ptrace_event_pid(PTRACE_EVENT_VFORK_DONE, pid);
            }

            put_pid(pid);
        } else {
            nr = PTR_ERR(p);
        }
        return nr;
}
```

### do_fork 流程

1. 调用 copy_process 为子进程复制出一份进程信息

2. 如果是 vfork 初始化完成处理信息

3. 调用 wake_up_new_task 将子进程加入调度器，为之分配 CPU

4. 如果是 vfork，父进程等待子进程完成 exec 替换自己的地址空间


### copy_process 流程

```
static struct task_struct *copy_process(unsigned long clone_flags,
                    unsigned long stack_start,
                    unsigned long stack_size,
                    int __user *child_tidptr,
                    struct pid *pid,
                    int trace)
{
    int retval;

    //创建进程描述符指针
    struct task_struct *p;

    //……

    //复制当前的 task_struct
    p = dup_task_struct(current);

    //……

    //初始化互斥变量   
    rt_mutex_init_task(p);

    //检查进程数是否超过限制，由操作系统定义
    if (atomic_read(&p->real_cred->user->processes) >=
            task_rlimit(p, RLIMIT_NPROC)) {
        if (p->real_cred->user != INIT_USER &&
            !capable(CAP_SYS_RESOURCE) && !capable(CAP_SYS_ADMIN))
            goto bad_fork_free;
    }

    //……

    //检查进程数是否超过 max_threads 由内存大小决定
    if (nr_threads >= max_threads)
        goto bad_fork_cleanup_count;

    //……

    //初始化自旋锁
    spin_lock_init(&p->alloc_lock);
    //初始化挂起信号
    init_sigpending(&p->pending);
    //初始化 CPU 定时器
    posix_cpu_timers_init(p);


    //……

    //初始化进程数据结构，并把进程状态设置为 TASK_RUNNING
    retval = sched_fork(clone_flags, p);

    //复制所有进程信息，包括文件系统、信号处理函数、信号、内存管理等
    if (retval)
        goto bad_fork_cleanup_policy;

    retval = perf_event_init_task(p);
    if (retval)
        goto bad_fork_cleanup_policy;
    retval = audit_alloc(p);
    if (retval)
        goto bad_fork_cleanup_perf;
    /* copy all the process information */
    shm_init_task(p);
    retval = copy_semundo(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_audit;
    retval = copy_files(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_semundo;
    retval = copy_fs(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_files;
    retval = copy_sighand(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_fs;
    retval = copy_signal(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_sighand;
    retval = copy_mm(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_signal;
    retval = copy_namespaces(clone_flags, p);
    if (retval)
        goto bad_fork_cleanup_mm;
    retval = copy_io(clone_flags, p);

    //初始化子进程内核栈
    retval = copy_thread(clone_flags, stack_start, stack_size, p);

    //为新进程分配新的 pid
    if (pid != &init_struct_pid) {
        retval = -ENOMEM;
        pid = alloc_pid(p->nsproxy->pid_ns_for_children);
        if (!pid)
            goto bad_fork_cleanup_io;
    }

    //设置子进程 pid 
    p->pid = pid_nr(pid);


    //……


    //返回结构体 p
    return p;
}
```

追踪copy_process 代码（部分）

1. 调用 dup\_task_struct 复制当前的 task_struct

2. 检查进程数是否超过限制

3. 初始化自旋锁、挂起信号、CPU 定时器等

4. 调用 sched_fork 初始化进程数据结构，并把进程状态设置为 TASK_RUNNING

5. 复制所有进程信息，包括文件系统、信号处理函数、信号、内存管理等

6. 调用 copy_thread 初始化子进程内核栈

7. 为新进程分配并设置新的pid


### dup\_task_struct 流程

```
static struct task_struct *dup_task_struct(struct task_struct *orig)
{
    struct task_struct *tsk;
    struct thread_info *ti;
    int node = tsk_fork_get_node(orig);
    int err;

    //分配一个 task_struct 节点
    tsk = alloc_task_struct_node(node);
    if (!tsk)
        return NULL;

    //分配一个 thread_info 节点，包含进程的内核栈，ti 为栈底
    ti = alloc_thread_info_node(tsk, node);
    if (!ti)
        goto free_tsk;

    //将栈底的值赋给新节点的栈
    tsk->stack = ti;

    //……

    return tsk;

}
```



1. 调用alloc\_task_struct_node分配一个 task_struct 节点

2. 调用alloc_thread_info_node分配一个 thread_info 节点，其实是分配了一个thread_union联合体,将栈底返回给 ti
	```
	union thread_union {
	   struct thread_info thread_info;
	  unsigned long stack[THREAD_SIZE/sizeof(long)];
	};
	```
3. 最后将栈底的值 ti 赋值给新节点的栈
最终执行完dup\_task_struct之后，子进程除了tsk->stack指针不同之外，全部都一样！


### sched_fork 流程

core.c

```
sched_forkcorint sched_fork(unsigned long clone_flags, struct task_struct *p)
{
    unsigned long flags;
    int cpu = get_cpu();

    __sched_fork(clone_flags, p);

    //将子进程状态设置为 TASK_RUNNING
    p->state = TASK_RUNNING;

    //……

    //为子进程分配 CPU
    set_task_cpu(p, cpu);

    put_cpu();
    return 0;
}
```



可以看到sched\_fork大致完成了两项重要工作：
1. 将子进程状态设置为 TASK_RUNNING
2. 为其分配 CPU


### copy_thread 流程
```
int copy_thread(unsigned long clone_flags, unsigned long sp,
    unsigned long arg, struct task_struct *p)
{
    //获取寄存器信息
    struct pt_regs *childregs = task_pt_regs(p);
    struct task_struct *tsk;
    int err;

    p->thread.sp = (unsigned long) childregs;
    p->thread.sp0 = (unsigned long) (childregs+1);
    memset(p->thread.ptrace_bps, 0, sizeof(p->thread.ptrace_bps));

    if (unlikely(p->flags & PF_KTHREAD)) {
        //内核线程
        memset(childregs, 0, sizeof(struct pt_regs));
        p->thread.ip = (unsigned long) ret_from_kernel_thread;
        task_user_gs(p) = __KERNEL_STACK_CANARY;
        childregs->ds = __USER_DS;
        childregs->es = __USER_DS;
        childregs->fs = __KERNEL_PERCPU;
        childregs->bx = sp; /* function */
        childregs->bp = arg;
        childregs->orig_ax = -1;
        childregs->cs = __KERNEL_CS | get_kernel_rpl();
        childregs->flags = X86_EFLAGS_IF | X86_EFLAGS_FIXED;
        p->thread.io_bitmap_ptr = NULL;
        return 0;
    }

    //将当前寄存器信息复制给子进程
    *childregs = *current_pt_regs();

    //子进程 eax 置 0，因此fork 在子进程返回0
    childregs->ax = 0;
    if (sp)
        childregs->sp = sp;

    //子进程ip 设置为ret_from_fork，因此子进程从ret_from_fork开始执行
    p->thread.ip = (unsigned long) ret_from_fork;

    //……

    return err;
}
```
copy_thread 这段代码为我们解释了两个相当重要的问题！
1. 为什么 fork 在子进程中返回0，原因是childregs->ax = 0;这段代码将子进程的 eax 赋值为0
2. p->thread.ip = (unsigned long) ret_from_fork;将子进程的 ip 设置为 ret_form_fork 的首地址，因此子进程是从 ret_from_fork 开始执行的

# 六、总结

创建一个新进程在内核中的执行过程大致如下:
1. 使用系统调用Sys\_clone(或fork,vfork)系统调用创建一个新进程，而且都是通过调用do_fork来实现进程的创建；
2. Linux通过复制父进程PCB的task_struct来创建一个新进程，要给新进程分配一个新的内核堆栈;
3. 要修改复制过来的进程数据，比如pid、进程链表等等执行copy_process和copy_thread
4. p->thread.sp = (unsigned long) childregs; //调度到子进程时的内核栈顶
5. p->thread.ip = (unsigned long) ret_from_fork; //调度到子进程时的第一条指令地址

# 参考资料

http://www.jianshu.com/p/a89f622e64ea

http://blog.csdn.net/renwotao2009/article/details/51472435

http://blog.csdn.net/sinat_34144680/article/details/51016244

http://blog.sina.com.cn/s/blog_6a0236970102vh6w.html

http://m.blog.csdn.net/article/details?id=51043739

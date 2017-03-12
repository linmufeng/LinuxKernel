原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求

使用gdb跟踪调试内核从start\_kernel到init进程启动，详细分析从start_kernel到init进程启动的过程。

# 二、实验内容

- 详细分析从start\_kernel到init进程启动的过程，内容围绕Linux内核的启动过程，即从start_kernel到init进程启动；
- 仔细分析start_kernel函数的执行过程
- 总结部分需要阐明自己对“Linux系统启动过程”的理解，尤其是idle进程、1号进程是怎么来的。

# 三、实验环境

实验楼linux内核分析课程线上虚拟机的linux环境

主要优点：环境免配置，使用方便，不消耗主机资源。


# 四、实验过程

## 1. 使用实验楼的虚拟机打开shell

```
cd LinuxKernel/
qemu -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img
```
![这里写图片描述](http://img.blog.csdn.net/20170312163922405?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

内核启动完成后将进入[menu程序](https://github.com/mengning/menu)（《软件工程C编码实践篇》的课程项目）。

![这里写图片描述](http://img.blog.csdn.net/20170312163941683?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 2. 使用gdb跟踪调试内核

```
qemu -kernel linux-3.18.6/arch/x86/boot/bzImage -initrd rootfs.img -s -S #关于-s和-S选项的说明：
# -S freeze CPU at startup (use ’c’ to start execution)
# -s shorthand for -gdb tcp::1234 若不想使用1234端口，则可以使用-gdb tcp:xxxx来取代-s选项
```

可以看到，内核启动中被冻结起来了，当前状态是Stopped。

![这里写图片描述](http://img.blog.csdn.net/20170312164031375?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)



另开一个shell窗口

![这里写图片描述](http://img.blog.csdn.net/20170312164101606?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

```
gdb
（gdb）file linux-3.18.6/vmlinux # 在gdb界面中targe remote之前加载符号表
（gdb）target remote:1234 # 建立gdb和gdbserver之间的连接,按c 让qemu上的Linux继续运行
（gdb）break start_kernel # 断点的设置可以在target remote之前，也可以在之后
```
![这里写图片描述](http://img.blog.csdn.net/20170312164151840?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

按住c回车，系统就可以继续执行了，会一直执行到start_kernel的位置

```
（gdb）c # 系统继续执行
```

![这里写图片描述](http://img.blog.csdn.net/20170312164638358?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)




可以看到，内核启动在start\_kernel进程处停住了，在这之前从Power On开始是很长的初始化过程。x86 CPU启动的第一个动作CS:EIP=FFFF:0000H（换算为物理地址为000FFFF0H，因为16位CPU有20根地址线)，即BIOS程序的位置，在这个过程计算机只是单纯地执行二进制指令，此时并不存在真正的进程。然后是运行到第二个断点rest_init,这是start_kernel调用的最后一个函数。设置相关的断点，这里我们将断定设在函数break start_kernel以及rest_init处。继续程序并观察断点处代码。

# 五、start_kernel函数的执行过程

系统进入start\_kernel这个函数之前已经进行了一些最低限度的初始化，再往前研究就涉及很多硬件相关及编程语言了。内核即进入了C语言部分，它完成了内核的大部分初始化工作。实际上，可以将start_kernel函数看做内核的main函数。 这个函数在init/main.c：

```
asmlinkage __visible void __init start_kernel(void)  
{  
    char *command_line;  
    char *after_dashes;  
  
    /* 
     * Need to run as early as possible, to initialize the 
     * lockdep hash: 
     */  
    lockdep_init();  
    set_task_stack_end_magic(&init_task);  
    smp_setup_processor_id();  
    debug_objects_early_init();
```

首先是调用 lockdep_init函数

```
void lockdep_init(void)  
{  
    int i;  
  
    /* 
     * Some architectures have their own start_kernel() 
     * code which calls lockdep_init(), while we also 
     * call lockdep_init() from the start_kernel() itself, 
     * and we want to initialize the hashes only once: 
     */  
    if (lockdep_initialized)  
        return;  
  
    for (i = 0; i < CLASSHASH_SIZE; i++)  
        INIT_LIST_HEAD(classhash_table + i);  
  
    for (i = 0; i < CHAINHASH_SIZE; i++)  
        INIT_LIST_HEAD(chainhash_table + i);  
  
    lockdep_initialized = 1;  
}  
```

在start\_kernel函数的最后调用了rest_init函数进行后续的初始化，生成了一号的进程。


```
static noinline void __init_refok rest_init(void)
{
    int pid;

    rcu_scheduler_starting();
    /*
     * We need to spawn init first so that it obtains pid 1, however
     * the init task will end up wanting to create kthreads, which, if
     * we schedule it before we create kthreadd, will OOPS.
     */
    kernel_thread(kernel_init, NULL, CLONE_FS);
    numa_default_policy();
    pid = kernel_thread(kthreadd, NULL, CLONE_FS | CLONE_FILES);
```


pid= kernel\_thread(kthreadd, NULL, CLONE\_FS | CLONE_FILES); 这行代码是创建一个干净内核线程，以便以后其它所有内核线程全部拷贝它，并由它来创建，这样达到更方便创建线程。

```
rcu_read_lock();
kthreadd_task = find_task_by_pid_ns(pid, &init_pid_ns);
rcu_read_unlock();
complete(&kthreadd_done);
/*
* The boot idle thread must execute schedule()
* at least once to get things moving:
*/
init_idle_bootup_task(current);
```

init\_idle\_bootup\_task(current); 这行代码是初始化空闲进程的调度器，以便让空闲进程知道怎么样调度任务列表里的进程。current是指向当前IDLE任务的结构。

schedule_preempt_disabled();
/* Call into cpu_idle with preempt disabled */
cpu_startup_entry(CPUHP_ONLINE);
     这行代码是调用进程调度函数schedule，主要初始化调度器可以切换回到空闲任务。并增加内核抢先计数。

# 六、总结

在本实验中，我分析了Linux系统的启动过程。最初执行的进程即是0号进程init\_task，它是被静态产生的，内存栈的位置固定,执行一些初始化的工作。一直到start\_kernel开始调用执行sched\_init()，0号进程被init\_idle(current, smp\_processor_id())进程初始化成为一个idle task,变成上一次实验中的进程一样的，通过一个while循环不断执行，只要运行栈里没有别的进程它就执行，循环中不断检测运行栈里是否有其他进程并通过schedule函数进行调度。

其中idle进程的产生为：idle是一个进程，其pid号为 0。其前身是系统创建的第一个进程，也是唯一一个没有通过fork()产生的进程。它在本实验中，具体是由init/main.c中start_kernel函数的set_task_stack_end_magic(&init_task)这一行开始实现的。其中的init_task就是手工创建的PCB，pid=0的进程，也就是最终的idle进程。

而1号进程的产生为：而到了kernel\_thread(kernel_init, NULL, CLONE_FS);则通过fork()建立了pid=1的1号进程，也叫init进程，它是第一个用户态进程，它会继续完成剩下的初始化工作，成为系统中的其他所有进程的祖先。而创建了1号进程后，随着init\_idle\_bootup\_task(current);等函数的调用，0号进程就演变成了idle进程。而idle进程就是当系统没有进程需要执行的时候来调度用的。所以start\_kernel里、rest\_init里创建了0号进程，该进程在系统初始化时候建立，并在系统运行过程中一直存在；而由0号进程，生成了1号进程，以及之后的许许多多的进程。最后进入了cpu_startup\_entry。这个其实就是调用了cpu\_idle。其实里面就是在while循环里调用了0号进程。

虽然在实验中我出不体会了Linux系统的启动过程，但是对Linux系统的理解还不深入，需要进一步加强。

原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求
- 使用gdb跟踪分析一个系统调用内核函数（您上周选择那一个系统调用），系统调用列表参见http://codelab.shiyanlou.com/xref/linux-3.18.6/arch/x86/syscalls/syscall_32.tbl ,推荐在实验楼Linux虚拟机环境下完成实验。

- 根据本周所学知识分析系统调用的过程，从system_call开始到iret结束之间的整个过程，并画出简要准确的流程图。


# 二、实验环境

本地虚拟机的linux环境（ubuntu12.02 32bit）

主要优点：使用方便，方便保存，不受网络影响。

# 三、实验过程

## 1.实验说明
本实验基于孟老师的github项目：[mykernel](https://github.com/mengning/mykernel)。

## 2.添加自定义代码

### 在LinuxKernel/menu/test.c中添加如图所示的代码

![这里写图片描述](http://img.blog.csdn.net/20170326211839057?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

运行自动编译脚本，生成根文件系统
```
make rootfs
```

### MenuOS的运行
![这里写图片描述](http://img.blog.csdn.net/20170326212121138?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170326212140685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 3.打开另一个shell终端，进入gdb调试工具

```
gdb

file linux-3.18.6/vmlinux //读入符号表

target remote:1234        //连接内核调试

b sys_getuid              //设置断点

c                         //继续执行

```
![这里写图片描述](http://img.blog.csdn.net/20170326220849125?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

进入MenuOS，调用执行getpid-asm

![这里写图片描述](http://img.blog.csdn.net/20170326221030346?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

![这里写图片描述](http://img.blog.csdn.net/20170326221009439?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

单步执行
![这里写图片描述](http://img.blog.csdn.net/20170327133134166?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

在system_call处设置断点，但是无法进入函数内部，指令运行结束，返回pid
![这里写图片描述](http://img.blog.csdn.net/20170327133219980?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 五、实验分析

中断相关的初始化代码是通过linux-3.18.6/init/main.c文件中的start\_kernel函数里的trap\_init()初始化的。执行int $0x80指令后内核开始执行system_call入口处开始的代码，位于entry_32.S汇编文件中。
下面分析system_call汇编代码：

```
中断相关的初始化代码是通过linux-3.18.6/init/main.c文件中的start_kernel函数里的trap_init()初始化的。执行int $0x80指令后内核开始执行system_call入口处开始的代码，位于entry_32.S汇编文件中。
下面分析system_call汇编代码：
```

\arch\x86\kernel\traps.c中的trap_init()

```
void __init trap_init(void)
{
    int i;
 
#ifdef CONFIG_EISA
    void __iomem *p = early_ioremap(0x0FFFD9, 4);
 
    if (readl(p) == 'E' + ('I'<<8) + ('S'<<16) + ('A'<<24))
        EISA_bus = 1;
    early_iounmap(p, 4);
#endif
 
    set_intr_gate(X86_TRAP_DE, divide_error);
    set_intr_gate_ist(X86_TRAP_NMI, &nmi, NMI_STACK);
    /* int4 can be called from all */
    set_system_intr_gate(X86_TRAP_OF, &overflow);
    set_intr_gate(X86_TRAP_BR, bounds);
    set_intr_gate(X86_TRAP_UD, invalid_op);
    set_intr_gate(X86_TRAP_NM, device_not_available);
#ifdef CONFIG_X86_32
    set_task_gate(X86_TRAP_DF, GDT_ENTRY_DOUBLEFAULT_TSS);
#else
    set_intr_gate_ist(X86_TRAP_DF, &double_fault, DOUBLEFAULT_STACK);
#endif
    set_intr_gate(X86_TRAP_OLD_MF, coprocessor_segment_overrun);
    set_intr_gate(X86_TRAP_TS, invalid_TSS);
    set_intr_gate(X86_TRAP_NP, segment_not_present);
    set_intr_gate(X86_TRAP_SS, stack_segment);
    set_intr_gate(X86_TRAP_GP, general_protection);
    set_intr_gate(X86_TRAP_SPURIOUS, spurious_interrupt_bug);
    set_intr_gate(X86_TRAP_MF, coprocessor_error);
    set_intr_gate(X86_TRAP_AC, alignment_check);
#ifdef CONFIG_X86_MCE
    set_intr_gate_ist(X86_TRAP_MC, &machine_check, MCE_STACK);
#endif
    set_intr_gate(X86_TRAP_XF, simd_coprocessor_error);
 
    /* Reserve all the builtin and the syscall vector: */
    for (i = 0; i < FIRST_EXTERNAL_VECTOR; i++)
        set_bit(i, used_vectors);
 
#ifdef CONFIG_IA32_EMULATION
    set_system_intr_gate(IA32_SYSCALL_VECTOR, ia32_syscall);
    set_bit(IA32_SYSCALL_VECTOR, used_vectors);
#endif
 
#ifdef CONFIG_X86_32
    set_system_trap_gate(SYSCALL_VECTOR, &system_call);
    set_bit(SYSCALL_VECTOR, used_vectors);
#endif
 
    /*
     * Set the IDT descriptor to a fixed read-only location, so that the
     * "sidt" instruction will not leak the location of the kernel, and
     * to defend the IDT against arbitrary memory write vulnerabilities.
     * It will be reloaded in cpu_init() */
    __set_fixmap(FIX_RO_IDT, __pa_symbol(idt_table), PAGE_KERNEL_RO);
    idt_descr.address = fix_to_virt(FIX_RO_IDT);
 
    /*
     * Should be a barrier for any external CPU state:
     */
    cpu_init();
 
    x86_init.irqs.trap_init();
 
#ifdef CONFIG_X86_64
    memcpy(&debug_idt_table, &idt_table, IDT_ENTRIES * 16);
    set_nmi_gate(X86_TRAP_DB, &debug);
    set_nmi_gate(X86_TRAP_BP, &int3);
#endif
}
```
第48行set\_system_trap_gate(SYSCALL_VECTOR, &system_call);设置了系统调用的中断向量（即系统调用的入口地址），即system_call“函数”的地址绑定到中断向量。所以在int $0x80之后，就转到中断向量，从而执行中断处理程序（系统调用程序）。也就是说中断向量指向的中断处理程序就是int $0x80指令执行完之后接着执行的指令的地方。

/arch/x86/kernel/entry_32.S

```
ENTRY(system_call)
    RING0_INT_FRAME         # can't unwind into user space anyway
    ASM_CLAC
    pushl_cfi %eax          # save orig_eax
    SAVE_ALL//保存现场
    GET_THREAD_INFO(%ebp)//获取thread_info结构的信息
                    # system call tracing in operation / emulation
    testl $_TIF_WORK_SYSCALL_ENTRY,TI_flags(%ebp)// 测试是否有系统跟踪
    jnz syscall_trace_entry//如果有系统跟踪，先执行，然后再回来
    cmpl $(NR_syscalls), %eax//比较eax中的系统调用号和最大syscall，超过则无效
    jae syscall_badsys//无效的系统调用 直接返回
syscall_call:
    call *sys_call_table(,%eax,4)//系统调用处理程序使用eax传过来的系统调用号查表来执行相应函数
syscall_after_call://保存返回值
    movl %eax,PT_EAX(%esp)      # store the return value
syscall_exit:
    LOCKDEP_SYS_EXIT
    DISABLE_INTERRUPTS(CLBR_ANY)    # make sure we don't miss an interrupt
                    # setting need_resched or sigpending
                    # between sampling and the iret
    TRACE_IRQS_OFF
    movl TI_flags(%ebp), %ecx
    testl $_TIF_ALLWORK_MASK, %ecx  # current->work//检测是否所有工作已完成
    jne syscall_exit_work//未完成，则去执行这些任务
 
restore_all:
    TRACE_IRQS_IRET
restore_all_notrace:
#ifdef CONFIG_X86_ESPFIX32
    movl PT_EFLAGS(%esp), %eax  # mix EFLAGS, SS and CS
    # Warning: PT_OLDSS(%esp) contains the wrong/random values if we
    # are returning to the kernel.
    # See comments in process.c:copy_thread() for details.
    movb PT_OLDSS(%esp), %ah
    movb PT_CS(%esp), %al
    andl $(X86_EFLAGS_VM | (SEGMENT_TI_MASK << 8) | SEGMENT_RPL_MASK), %eax
    cmpl $((SEGMENT_LDT << 8) | USER_RPL), %eax
    CFI_REMEMBER_STATE
    je ldt_ss           # returning to user-space with LDT SS
#endif
restore_nocheck://恢复现场
    RESTORE_REGS 4          # skip orig_eax/error_code
irq_return:
    INTERRUPT_RETURN
.section .fixup,"ax"
```

SAVE_ALL
```
.macro SAVE_ALL
    cld
    PUSH_GS
    pushl_cfi %fs
    /*CFI_REL_OFFSET fs, 0;*/
    pushl_cfi %es
    /*CFI_REL_OFFSET es, 0;*/
    pushl_cfi %ds
    /*CFI_REL_OFFSET ds, 0;*/
    pushl_cfi %eax
    CFI_REL_OFFSET eax, 0
    pushl_cfi %ebp
    CFI_REL_OFFSET ebp, 0
    pushl_cfi %edi
    CFI_REL_OFFSET edi, 0
    pushl_cfi %esi
    CFI_REL_OFFSET esi, 0
    pushl_cfi %edx
    CFI_REL_OFFSET edx, 0
    pushl_cfi %ecx
    CFI_REL_OFFSET ecx, 0
    pushl_cfi %ebx
    CFI_REL_OFFSET ebx, 0
    movl $(__USER_DS), %edx
    movl %edx, %ds
    movl %edx, %es
    movl $(__KERNEL_PERCPU), %edx
    movl %edx, %fs
    SET_KERNEL_GS %edx
.endm
```



在这段代码中，保存了相关寄存器的值。它们是：ES,DS,EAX,EBP,EDI,ESI,EDX,ECX,EBX等等。从这里寄存器的顺序可以知道压栈的最后压入的是ebx，这里压入的栈是内核栈。

RESTORE_REGS

```
.macro RESTORE_INT_REGS
    popl_cfi %ebx
    CFI_RESTORE ebx
    popl_cfi %ecx
    CFI_RESTORE ecx
    popl_cfi %edx
    CFI_RESTORE edx
    popl_cfi %esi
    CFI_RESTORE esi
    popl_cfi %edi
    CFI_RESTORE edi
    popl_cfi %ebp
    CFI_RESTORE ebp
    popl_cfi %eax
    CFI_RESTORE eax
.endm
 
.macro RESTORE_REGS pop=0
    RESTORE_INT_REGS
1:  popl_cfi %ds
    /*CFI_RESTORE ds;*/
2:  popl_cfi %es
    /*CFI_RESTORE es;*/
3:  popl_cfi %fs
    /*CFI_RESTORE fs;*/
    POP_GS \pop
.pushsection .fixup, "ax"
4:  movl $0, (%esp)
    jmp 1b
5:  movl $0, (%esp)
    jmp 2b
6:  movl $0, (%esp)
    jmp 3b
.popsection
    _ASM_EXTABLE(1b,4b)
    _ASM_EXTABLE(2b,5b)
    _ASM_EXTABLE(3b,6b)
    POP_GS_EX
.endm
```

从上面代码中可以看出SAVE_ALL和RESTORE_INT_REGS是对应的

源码几百行过于复杂，孟老师将它简化几十行的汇编伪代码：

```
.macro INTERRUPT_RETURN
         iret
.endm 
.marco  SAVE_ALL
 
 ...
.endm
 .macro RESTORE_INT_REGS
     ...
 .endm
ENTRY（system_call）
     SAVE_ALL
 syscall_call:
     call *sys_call_table(%eax,4)
     movl %eax,PT_EAX(%esp)  #store the return value
 syscall_exit:
     testl $_TLF_ALLWORK_MASK,%ecx  #current->work//检查是否还有其它工作要完成
     jne syscall_exit_work
restore_all://恢复处理器状态
    RESTORE_INT_REGS
irq_return:
    INTERRUPT_RETURN
ENDPROC(system_call)
syscall_exit_work:
    testl $_TIF_WORK_SYSCALL_EXIT,%ecx//测试syscall退出信号
     jz work_pending //还有其它工作要做
 END(syscall_exit_work)
 work_pending:
     testb $_TIF_NEED_RESCHED,%cl //检查是否需要重新调度
     jz work_notifysig //不需要重新调度
 work_resched://需要重新调度
     call schedule //调度进程
      jz retore_all//没有其它的事,则恢复处理器状态
  work_notifysig:   #deal with pending signals//处理挂起的信号
  ...
  END(work_pending)
  
```

系统调用流程图

![这里写图片描述](http://img.blog.csdn.net/20170326225135343?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


# 六、总结

1. 执行int 0x80指令后系统从用户态进入内核态，跳到system\_call()函数处执行相应服务进程。在此过程中内核先保存中断环境，然后执行系统调用函数。
2. system\_call()函数通过系统调用号查找系统调用表sys_cal_table来查找具体系统调用服务进程。
3. 执行完系统调用后，iret之前，内核会检查是否有新的中断产生、是否需要进程切换、是否学要处理其它进程发送过来的信号等。 
4. 内核是处理各种系统调用的中断集合，通过中断机制实现进程上下文的切换，通过系统调用管理整个计算机软硬件资源。
5.如没有新的中断，restore保存的中断环境并返回用户态完成一个系统调用过程。

系统调用本质上是发生了一次软件中断。

# 参考资料

- [分析system_call中断处理过程 1](http://blog.csdn.net/myfather103/article/details/44700523)

- [分析system_call中断处理过程 2](http://www.cnblogs.com/jorilee/p/5326670.html)

- [分析system_call中断处理过程 3](https://xuezhaojiang.github.io/LinuxCore/lab5/lab5.html)

- [内核抢占与中断返回](http://www.cnblogs.com/hustcat/archive/2009/08/31/1557507.html)

- [说说中断上下文的切换](http://blog.csdn.net/zhaolianxun1987/article/details/51236216)

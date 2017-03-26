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

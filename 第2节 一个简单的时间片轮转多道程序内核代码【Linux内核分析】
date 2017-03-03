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

# 五、linux内核代码分析

# 六、实验知识点回顾

# 七、实验总结

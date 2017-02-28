原创作品转载请注明出处 +《Linux内核分析》MOOC课程http://mooc.study.163.com/course/USTC-1000029000

# 一、实验要求

　　实验部分（以下命令为实验楼64位Linux虚拟机环境下适用，32位Linux环境可能会稍有不同）

写一个简单的C程序，将其编译成汇编代码，并分析汇编代码的工作过程中堆栈的变化。

# 二、实验过程
 
## 1、写一个简单的C程序

> main.c

![main.c](http://img.blog.csdn.net/20170221144416213?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2、将C程序编译成汇编代码
使用下面的命令
```
gcc –S –o main.s main.c -m32 命令编译成汇编代码
```
　　将C程序编译成汇编代码，将所有开头带.的代码删除

> main.s

![main.s](http://img.blog.csdn.net/20170221144438260?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 三、实验分析

## 1、主函数工作过程中堆栈的变化
![这里写图片描述](http://img.blog.csdn.net/20170221152734191?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 2、f(int x)函数工作过程中堆栈的变化
![这里写图片描述](http://img.blog.csdn.net/20170221181925111?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


## 3、g(int x)函数工作过程中堆栈的变化
![这里写图片描述](http://img.blog.csdn.net/20170221182122282?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

　　最后通过leave、ret，返回到main中，eax加48，并返回。


# 四、实验总结

目前计算机的工作模型基本上都是来自或改进于冯诺依曼体系结构。

1. 从硬件角度看，这种存储程序计算机工作模型由两大部分硬件组成，CPU和内存，总线连接两个硬件，负责信息的传递。

	这里引用老师上课所画的示意图：
	
 ![这里写图片描述](http://img.blog.csdn.net/20170223103631795?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvcXE0NzA4Njk4NTI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

2.  从程序员角度来看，CPU就是一直不停的工作，只要有指令产生，就依次执行，内存中存放着各种指令和数据，总线将两者相连接

3. 为什么需要掌握汇编代码
	　　在计算机中的数据都是由二进制的01串组成，进行01字符串的编码难度极大，高级语言的产生对底层进行屏蔽，只需要掌握基本的语法就能编程，然而却无法更加深刻地理解计算机底层的工作原理，因此，在第一次实验中，我们学习了如何反汇编一个简单的C程序，通过一行一行细致的解读汇编代码，对于计算机的工作模型有了一个较为感性的认识，特别是程序在内存中的状态和工作方式。

4. 汇编指令的执行
　　在汇编程序中，每一个函数就有一个函数栈，里面存放着汇编指令，CPU会首先从main函数中根据指令一条一条地执行，各种寄存器会紧密合作。遇到需要跳转访问子函数堆栈，首先做的工作就是保存当前指令的下一个位置，为了能够在子函数执行完后还能够继续当前函数指令的执行。用eax收集函数返回值，以便对于需要子函数的返回值时候，让函数从寄存器中去读取。

> <b>总结
> 
> - 采用冯诺依曼体系结构，CPU通过总线与内存连接，CPU中有IP负责执行语句并移动至下一个指令，内存中存放着具体指令和数据。
> - 寄存器分为通用寄存器和段寄存器，通用寄存器负责记录累加值、基地址、堆栈基地址与顶地址、变址、计数值等，段寄存器负责记录代码段、数据段、附加段和堆栈段。
> - IP指向代码段寄存器，随着逐条执行IP，代码段寄存器（CS）中的指令被一一执行，而CS也可以影响IP的指向，从而更改执行的流程，组成复杂的控制结构。


---
title: 操作系统笔记1
date: 2025-02-12 09:00
toc: true
---
这里的"操作系统笔记1",是指以蒋老师的![课程讲义](https://jyywiki.cn/)来进行预习、学习、复习、总结的记录，这里的1是指第一周的课程，即周二周四总共两次课，每次课是两节课，每节课是一个小时,总共16周课程。
我会花16周的时间来写这个笔记，基本上按照授课这个节奏来写，也可能因为各种事情实在抽不出时间就会顺延一次，笔记包括了一些take-away的知识点及我没有理解的知识点，在这里也用作记事本记录一下。
将使用2024春季学期的版本作为参考。
任何程序 = minimal.S = 状态机
这里的minimal.S是指用汇编语言编写的最小Helloworld程序
```c
#include <sys/syscall.h>
// The x86-64 system call Application Binary Interface (ABI):
//     System call number: RAX
//     Arguments: RDI, RSI, RDX, RCX, R8, R9
//     Return value: RAX
// See also: syscall(2) syscalls(2)

#define syscall3(id, a1, a2, a3) \
    movq $SYS_##id, %rax; \
    movq $a1, %rdi; \
    movq $a2, %rsi; \
    movq $a3, %rdx; \
    syscall

#define syscall2(id, a1, a2)  syscall3(id, a1, a2, 0)
#define syscall1(id, a1)  syscall2(id, a1, 0)

.globl _start
_start:
    syscall3(write, 1, addr1, addr2 - addr1)
    syscall1(exit, 1)

addr1:
    .ascii "\033[01;31mHello, OS World\033[0m\n"
addr2:
```
总是从被操作系统加载开始
通过另一个进程执行 execve 设置为初始状态
经历状态机执行 (计算 + syscalls)
进程管理：fork, execve, exit, ...
文件/设备管理：open, close, read, write, ...
存储管理：mmap, brk, ...
最终调用 _exit (exit_group) 退出

编译：是将高级语言转化为汇编语言的过程。

编译器的输入: 高级语言 (C) 代码 = 状态机
编译器的输出: 汇编代码 (指令序列) = 状态机
编译器 = 状态机之间的翻译器

Everything (高级语言代码、机器代码) 都是状态机；而编译器实现了两种状态机之间的翻译。无论何种状态机，在没有操作系统时，它们只能做纯粹的计算，甚至都不能把结果传递到程序之外——而程序与操作系统沟通的唯一桥梁是系统调用 (例如 x86-64 的 syscall 指令)。如此重要的桥梁，操作系统中自然也有工具：strace 可以查看程序运行过程中的系统调用序列

这次我主要学习的是内联汇编(c语言中嵌入汇编语言)和基础命令的组合使用，本次进行了学习，对于开发者来说，工具的组合是非常重要的。
strace会查看gcc的编译过程中所进行的系统调用过程，并将结果通过vim打开
strace -f gcc a.c | vim -
在vim编辑器里还可以 %!grep 进行查找

M1:
我们学习 UNIX/Linux 是如何把操作系统的状态放在文件系统中的。
以及我们如何读取这个文件系统中的进程列表和进程之间的父子关系。
每个正在运行的进程在 /proc 下都有一个以其 PID（进程 ID）命名的子目录（如 /proc/1234）。
查看proc文档：
man 5 proc

/proc/[PID]/status：进程状态（运行、睡眠等）。

/proc/[PID]/cmdline：启动进程的命令行参数。

/proc/[PID]/exe：指向进程的可执行文件。

/proc/[PID]/fd/：包含进程打开的文件描述符。

/proc/cpuinfo：CPU 的详细信息。

/proc/meminfo：内存使用情况。

/proc/version：内核版本信息。

/proc/filesystems：系统支持的文件系统类型。

/proc/net/：网络相关的信息（如 TCP/UDP 连接、路由表等）。

/proc/interrupts：中断信息。

/proc/ioports：I/O 端口信息。

/proc/scsi/：SCSI 设备信息。

内核参数调整：
/proc/sys/ 目录下的文件用于查看和修改内核参数（运行时配置）。
/proc/sys/net/ipv4/ip_forward：控制 IP 转发功能。
/proc/sys/kernel/hostname：系统主机名。

大多数文件是只读的（如 /proc/cpuinfo），但部分文件可写（如 /proc/sys/ 下的文件），用于调整内核参数。

/proc 中的文件并不是真实的磁盘文件，而是由内核动态生成的，内容会随系统状态变化而更新。

procfs 是用户空间程序与内核交互的重要接口，许多系统工具（如 ps、top）依赖 /proc 获取信息。
通过读取 /proc/meminfo、/proc/stat 等文件，监控系统资源使用情况。
通过 /proc/[PID]/ 目录下的文件，查看进程的运行状态、内存映射、打开的文件等。
通过修改 /proc/sys/ 下的文件，调整内核参数以优化系统性能。

编译和测试自动化非常重要。常见的 C 项目组织是编写 Makefile，在命令行中使用 make 实现编译，make test 完成测试。
会节省很多时间和精力。
只有用 make 命令编译，才会留下你的开发历史——实验的编译迟早会复杂到无法 “手工键入命令” 完成。
我们非常鼓励你在 IDE (例如 vscode) 中配置好 build script，这样可以一键编译/调试/运行。

以下两点有助于调试时放平心态：
(1) 机器永远是对的；
(2) 未测代码永远是错的。

在命令的输出的情况下新增表头，这里以ps命令为例:
ps -ef | head -n1 && ps -ef | grep bash


当使用man去查看相关函数的帮助时候，发现没有手册，需要手动安装
sudo apt search manpages
sudo apt install manpages 
sudo apt install manpages-dev
sudo apt install manpages-posix
sudo apt install manpages-posix-dev

读取文件夹和读取文件
readdir

## 重新复习了printf format的格式
int sprintf(char *str, const char *format, ...);

printf是将format字符串进行格式化并打印到标准输出，sprintf是将format字符串进行格式化并写到字符串str中。
%[flag][min width][precision][length modifier][conversion specifier]
对于precision精确度
.precision
对于f则是控制浮点数小数点后的几位数字，小数点后少则补0,多则四舍五入
对于d则是控制整数的至少的位数，少则补0，多则显示完整
对于s则是控制字符串的最大打印长度
对于g则是控制有效位数


对于这个简易版的pstree我们通过读取/proc文件夹和文件/proc/[pid]/status获取进程号、父进程号、进程名称等信息，
那么我如何构建一颗有向的多叉树？
当你每读取到一个(childId,parentId),如何插入到这个树中呢？
最终得到类似于这样的字符串 1(2(3(5,7),4(6)),8)


## 5个遗留问题
1. 你的程序遵守 POSIX 的返回值规定吗？如果你的 main 函数返回了非 0 的数值，我们将认为程序报告了错误——在非法的输入上返回 0，以及在合法的输入上返回非 0 都将导致 Wrong Answer。
2. 程序够 robust 吗？它会不会在一些非法的输入上 crash？如果系统里的进程很多呢？如果内存不够了呢？如果 open 或者 malloc 失败了呢？要知道，crash 一般是因为 undefined behavior (UB) 导致的——UB 没把所有的文件都删掉真是谢天谢地了。
3. 万一我得到进程号以后，进去发现文件没了 (进程终止了)，怎么办？会不会有这种情况？万一有我的程序会不会 crash……？
4. 进程的信息一直在变，文件的内容也一直在变 (两次 cat 的结果不同)。那我会不会读到不一致的信息 (前一半是旧信息、新一半是新信息)？这两个问题都是 race condition 导致的；我们将会在并发部分回到这个话题。
5. 如果我不确信这些事会不会发生，我有没有写一个程序，至少在压力环境下测试一下它们有没有可能发生？嗯，如果我同时运行很多程序，每个程序都不断扫描目录、读取文件，也观察不到这个问题，至少应该可以放点心。


相关材料:
https://gcc.gnu.org/onlinedocs/gcc/Extended-Asm.html
https://en.wikipedia.org/wiki/Volatile_(computer_programming)
https://cil-project.github.io/cil/
https://gitlab.com/zsaleeba/picoc
https://www.usenix.org/conference/osdi22/presentation/zhong
https://dl.acm.org/doi/abs/10.1145/3552326.3587458
http://www.columbia.edu/cu/computinghistory/vt100.html
https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html
https://www.gnu.org/software/libc/manual/html_node/Process-Identification.html
https://www.gnu.org/software/libc/manual/html_node/POSIX-Safety-Concepts.html
https://www.gnu.org/software/libc/manual/html_node/Accessing-Directories.html
https://www.cs.fsu.edu/~myers/c++/notes/c_io.html
https://www.cprogramming.com/tutorial/printf-format-strings.html

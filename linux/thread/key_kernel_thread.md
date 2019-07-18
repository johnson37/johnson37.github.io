# Key Kernel Thread
- init: this is the first process
- kthreadd: this thread aims to create other kernel thread
- ksoftirqd: this thread aims to handle software irqs.
- kworker: this thread aims to work the bottom of interrupts.
- migration: this thread aims to transfer some work from one CPU to another CPU.
- watchdog: watchdog can reset the system if the system doesn't work fine.
- khelper：这种内核线程只有一个，主要作用是指定用户空间的程序路径和环境变量, 最终运行指定的user space的程序，属于关键线程，不能关闭

# Interrupt
无论软中断，tasklet还是worker queue都是对中断下半部的处理方式。
通常情况下，硬中断由外部事件进行触发，我们在处理硬中断过程中，会先关闭硬中断，完成中断处理函数，重新打开硬中断。
考虑到，事件是异步的，系统应该尽可能短的关闭中断，即要求尽可能快的完成中断处理函数。当中断内需要处理的事情确实
很多的时候，中断被分成了两部分，中断上半部和中断的下半部。

在系统运行中，硬中断触发，我们在中断处理函数中完成中断上半部，重新开启硬中断。把中断下半部未完成的中断处理内容，
交给softirq，tasklet，或者kworker来处理。
## Basic Introduction
- Softirq:软中断。软中断可以被硬件中断打断，但是不可睡眠或者重新调度。
- tasklet:认为是软中断的一种。
- Kworker:kworker机制是一个内核线程，允许睡眠以及重新调度。
相对来说，softirq和tasklet用来处理相对实时性要求高一点的，而worker用来处理实时性要求低的。


## Softirq
### Definition
```c
struct softirq_action
{
    void    (*action)(struct softirq_action *);
};

enum
{
    HI_SOFTIRQ=0,
    TIMER_SOFTIRQ,
    NET_TX_SOFTIRQ,
    NET_RX_SOFTIRQ,
    BLOCK_SOFTIRQ,
    BLOCK_IOPOLL_SOFTIRQ,
    TASKLET_SOFTIRQ,
    SCHED_SOFTIRQ,
    HRTIMER_SOFTIRQ, /* Unused, but kept as tools rely on the
                numbering. Sigh! */
    RCU_SOFTIRQ,    /* Preferable RCU should always be the last softirq */
    
    NR_SOFTIRQS
};

```

### Register Interrupt Function
```c
void open_softirq(int nr, void (*action)(struct softirq_action *))
{
    softirq_vec[nr].action = action;
}

static int __init net_dev_init(void)
{  
    open_softirq(NET_TX_SOFTIRQ, net_tx_action);
    open_softirq(NET_RX_SOFTIRQ, net_rx_action);
}

```

### Trigger Software Interrupt
```c
void raise_softirq(unsigned int nr)
{
    unsigned long flags;

    local_irq_save(flags);
    raise_softirq_irqoff(nr);
    local_irq_restore(flags);
}
inline void raise_softirq_irqoff(unsigned int nr)
{
    __raise_softirq_irqoff(nr);

    /*
     * If we're in an interrupt or softirq, we're done
     * (this also catches softirq-disabled code). We will
     * actually run the softirq once we return from
     * the irq or softirq.
     *
     * Otherwise we wake up ksoftirqd to make sure we
     * schedule the softirq soon.
     */
    if (!in_interrupt())
        wakeup_softirqd();
}   

void __raise_softirq_irqoff(unsigned int nr)
{
    trace_softirq_raise(nr);
    or_softirq_pending(1UL << nr);
}

```
```c
//Above code is to raise the softirq, the real process for softirq exists in softirqd thread
static void run_ksoftirqd(unsigned int cpu)
{
    local_irq_disable();
    if (local_softirq_pending()) {
        /*
         * We can safely run softirq on inline stack, as we are not deep
         * in the task stack here.
         */
        __do_softirq();
        local_irq_enable();
        cond_resched_rcu_qs();
        return;
    }
    local_irq_enable();
}

asmlinkage __visible void __do_softirq(void)
{
    unsigned long end = jiffies + MAX_SOFTIRQ_TIME;
    unsigned long old_flags = current->flags;
    int max_restart = MAX_SOFTIRQ_RESTART;
    struct softirq_action *h;
    bool in_hardirq;
    __u32 pending;
    int softirq_bit;

    /*
     * Mask out PF_MEMALLOC s current task context is borrowed for the
     * softirq. A softirq handled such as network RX might set PF_MEMALLOC
     * again if the socket is related to swap
     */
    current->flags &= ~PF_MEMALLOC;

    pending = local_softirq_pending();
    account_irq_enter_time(current);

    __local_bh_disable_ip(_RET_IP_, SOFTIRQ_OFFSET);
    in_hardirq = lockdep_softirq_start();

restart:
    /* Reset the pending bitmask before enabling irqs */
    set_softirq_pending(0);

    local_irq_enable();

    h = softirq_vec;

    while ((softirq_bit = ffs(pending))) {
        unsigned int vec_nr;
        int prev_count;

        h += softirq_bit - 1;
        vec_nr = h - softirq_vec;
        prev_count = preempt_count();

        kstat_incr_softirqs_this_cpu(vec_nr);

        trace_softirq_entry(vec_nr);
        h->action(h);
        trace_softirq_exit(vec_nr);
        if (unlikely(prev_count != preempt_count())) {
            pr_err("huh, entered softirq %u %s %p with preempt_count %08x, exited with %08x?\n",
                   vec_nr, softirq_to_name[vec_nr], h->action,
                   prev_count, preempt_count());
            preempt_count_set(prev_count);
        }
        h++;
        pending >>= softirq_bit;
    }

    rcu_bh_qs();
    local_irq_disable();

    pending = local_softirq_pending();
    if (pending) {
        if (time_before(jiffies, end) && !need_resched() &&
            --max_restart)
            goto restart;

        wakeup_softirqd();
    }

    lockdep_softirq_end(in_hardirq);
    account_irq_exit_time(current);
    __local_bh_enable(SOFTIRQ_OFFSET);
    WARN_ON_ONCE(in_interrupt());
    tsk_restore_flags(current, old_flags, PF_MEMALLOC);
}

```

## Tasklet
Tasklet是一种软中断。Tasklet跟软中断的区别是：
- 软中断支持不同CPU并行执行。不管是同类型的还是不同类型的。
- tasklet仅仅支持不同类型的在不同CPU上执行，同种类型的不支持。
所以，对于软中断函数，需要保证函数可重入。而Tasklet则不需要保证。

```c
static inline void tasklet_schedule(struct tasklet_struct *t)
{
    if (!test_and_set_bit(TASKLET_STATE_SCHED, &t->state))
        __tasklet_schedule(t);
}
   
void __tasklet_schedule(struct tasklet_struct *t)
{
    unsigned long flags;
        
    local_irq_save(flags);
    t->next = NULL;
    *__this_cpu_read(tasklet_vec.tail) = t;
    __this_cpu_write(tasklet_vec.tail, &(t->next));
    raise_softirq_irqoff(TASKLET_SOFTIRQ);
    local_irq_restore(flags);
}

void tasklet_init(struct tasklet_struct *t,
          void (*func)(unsigned long), unsigned long data)
{
    t->next = NULL;
    t->state = 0;
    atomic_set(&t->count, 0);
    t->func = func;
    t->data = data;
}

```

```c
//Basic Example for usage 

#include <linux/module.h>
#include <linux/init.h>
#include <linux/fs.h>
#include <linux/kdev_t.h>
#include <linux/cdev.h>
#include <linux/kernel.h>
#include <linux/interrupt.h>  

static struct tasklet_struct my_tasklet;  
static void tasklet_handler (unsigned long data)
{
        printk(KERN_ALERT "tasklet_handler is running.\n");
}  

static int __init test_init(void)
{
        tasklet_init(&my_tasklet, tasklet_handler, 0);
        tasklet_schedule(&my_tasklet);
        return 0;
}  

static void __exit test_exit(void)
{
        tasklet_kill(&my_tasklet);
        printk(KERN_ALERT "test_exit running.\n");
}
MODULE_LICENSE("GPL");  

module_init(test_init);
module_exit(test_exit);

```
## Kworker

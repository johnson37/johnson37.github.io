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
上面部分实现只是把tasklet加到列表当中。

```c
void __init softirq_init(void)
{   
    int cpu;

    for_each_possible_cpu(cpu) {
        per_cpu(tasklet_vec, cpu).tail =
            &per_cpu(tasklet_vec, cpu).head;
        per_cpu(tasklet_hi_vec, cpu).tail =
            &per_cpu(tasklet_hi_vec, cpu).head;
    }   
            
    open_softirq(TASKLET_SOFTIRQ, tasklet_action);
    open_softirq(HI_SOFTIRQ, tasklet_hi_action);
}

//Here, we note that when t->counter is zero, it means enable this tasklet.
static void tasklet_action(struct softirq_action *a)
{
    struct tasklet_struct *list;

    local_irq_disable();
    list = __this_cpu_read(tasklet_vec.head);
    __this_cpu_write(tasklet_vec.head, NULL);
    __this_cpu_write(tasklet_vec.tail, this_cpu_ptr(&tasklet_vec.head));
    local_irq_enable();

    while (list) {
        struct tasklet_struct *t = list;

        list = list->next;

        if (tasklet_trylock(t)) {
            if (!atomic_read(&t->count)) {
                if (!test_and_clear_bit(TASKLET_STATE_SCHED,
                            &t->state))
                    BUG();
                t->func(t->data);
                tasklet_unlock(t);
                continue;
            }
            tasklet_unlock(t);
        }

        local_irq_disable();
        t->next = NULL;
        *__this_cpu_read(tasklet_vec.tail) = t;
        __this_cpu_write(tasklet_vec.tail, &(t->next));
        __raise_softirq_irqoff(TASKLET_SOFTIRQ);
        local_irq_enable();
    }
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
kworker也是为了完成中断的下半部，与softirq以及tasklet不同的是，kworker允许休眠和被调度，所以相对来说，实时性会差一些。

### Code Flow

#### Init Kworkers
```c
static noinline void __init kernel_init_freeable(void)
{
    /*
     * Wait until kthreadd is all set-up.
     */
    wait_for_completion(&kthreadd_done);

    /* Now the scheduler is fully set up and can do blocking allocations */
    gfp_allowed_mask = __GFP_BITS_MASK;

    /*
     * init can allocate pages on any node
     */
    set_mems_allowed(node_states[N_MEMORY]);
    /*
     * init can run on any cpu.
     */
    set_cpus_allowed_ptr(current, cpu_all_mask);

    cad_pid = task_pid(current);

    smp_prepare_cpus(setup_max_cpus);

    do_pre_smp_initcalls(); //Here we will init workqueue and create some kworker thread.
    lockup_detector_init();

    smp_init();
    sched_init_smp();

    page_alloc_init_late();

    do_basic_setup();

    /* Open the /dev/console on the rootfs, this should never fail */
    if (sys_open((const char __user *) "/dev/console", O_RDWR, 0) < 0)
        pr_err("Warning: unable to open an initial console.\n");

    (void) sys_dup(0);
    (void) sys_dup(0);
	...
}

static int __init init_workqueues(void)
{
    /* initialize CPU pools */
    for_each_possible_cpu(cpu) {
        struct worker_pool *pool;

        i = 0;
        for_each_cpu_worker_pool(pool, cpu) {
            BUG_ON(init_worker_pool(pool));
            pool->cpu = cpu;
            cpumask_copy(pool->attrs->cpumask, cpumask_of(cpu));
            pool->attrs->nice = std_nice[i++];
            pool->node = cpu_to_node(cpu);

            /* alloc pool ID */
            mutex_lock(&wq_pool_mutex);
            BUG_ON(worker_pool_assign_id(pool));
            mutex_unlock(&wq_pool_mutex);
        }
    }

    /* create the initial worker */
    for_each_online_cpu(cpu) {
        struct worker_pool *pool;

        for_each_cpu_worker_pool(pool, cpu) {
            pool->flags &= ~POOL_DISASSOCIATED;
            BUG_ON(!create_worker(pool));
        }
    }
    /* create default unbound and ordered wq attrs */
    for (i = 0; i < NR_STD_WORKER_POOLS; i++) {
        struct workqueue_attrs *attrs;

        BUG_ON(!(attrs = alloc_workqueue_attrs(GFP_KERNEL)));
        attrs->nice = std_nice[i];
        unbound_std_wq_attrs[i] = attrs;

        /*
         * An ordered wq should have only one pwq as ordering is
         * guaranteed by max_active which is enforced by pwqs.
         * Turn off NUMA so that dfl_pwq is used for all nodes.
         */
        BUG_ON(!(attrs = alloc_workqueue_attrs(GFP_KERNEL)));
        attrs->nice = std_nice[i];
        attrs->no_numa = true;
        ordered_wq_attrs[i] = attrs;
    }

}

static struct worker *create_worker(struct worker_pool *pool)
{
    struct worker *worker = NULL;
    int id = -1;
    char id_buf[16];

    /* ID is needed to determine kthread name */
    id = ida_simple_get(&pool->worker_ida, 0, 0, GFP_KERNEL);
    if (id < 0)
        goto fail;

    worker = alloc_worker(pool->node);
    if (!worker)
        goto fail;

    worker->pool = pool;
    worker->id = id;

    if (pool->cpu >= 0)
        snprintf(id_buf, sizeof(id_buf), "%d:%d%s", pool->cpu, id,
             pool->attrs->nice < 0  ? "H" : "");
    else
        snprintf(id_buf, sizeof(id_buf), "u%d:%d", pool->id, id);

    worker->task = kthread_create_on_node(worker_thread, worker, pool->node,
                          "kworker/%s", id_buf);

    /* successful, attach the worker to the pool */
    worker_attach_to_pool(worker, pool);

    /* start the newly created worker */
    spin_lock_irq(&pool->lock);
    worker->pool->nr_workers++;
    worker_enter_idle(worker);
    wake_up_process(worker->task);
    spin_unlock_irq(&pool->lock);

    return worker;

}

```
#### How these worker works?
```c
static int worker_thread(void *__worker)
{
    struct worker *worker = __worker;
    struct worker_pool *pool = worker->pool;
    do {
        struct work_struct *work =
            list_first_entry(&pool->worklist,
                     struct work_struct, entry);

        if (likely(!(*work_data_bits(work) & WORK_STRUCT_LINKED))) {
            /* optimization path, not strictly necessary */
            process_one_work(worker, work);
            if (unlikely(!list_empty(&worker->scheduled)))
                process_scheduled_works(worker);
        } else {
            move_linked_works(work, &worker->scheduled, NULL);
            process_scheduled_works(worker);
        }
    } while (keep_working(pool));
}
static void process_one_work(struct worker *worker, struct work_struct *work)
__releases(&pool->lock)
__acquires(&pool->lock)
{
	...
	/* claim and dequeue */
    debug_work_deactivate(work);
    hash_add(pool->busy_hash, &worker->hentry, (unsigned long)work);
    worker->current_work = work;
    worker->current_func = work->func;
    worker->current_pwq = pwq;
    work_color = get_work_color(work);

    worker->current_func(work);
	...
}
```

#### How to add one task to the kworker
- Init work
```c
#define __INIT_WORK(_work, _func, _onstack)             \
    do {                                \
        static struct lock_class_key __key;         \
                                    \
        __init_work((_work), _onstack);             \
        (_work)->data = (atomic_long_t) WORK_DATA_INIT();   \
        lockdep_init_map(&(_work)->lockdep_map, #_work, &__key, 0); \
        INIT_LIST_HEAD(&(_work)->entry);            \
        (_work)->func = (_func);                \
    } while (0)
```
- Add to Workqueue
```c
static inline bool schedule_work(struct work_struct *work)
{
    return queue_work(system_wq, work);
}

static inline bool queue_work(struct workqueue_struct *wq,
                  struct work_struct *work)
{
    return queue_work_on(WORK_CPU_UNBOUND, wq, work);
}

bool queue_work_on(int cpu, struct workqueue_struct *wq,
           struct work_struct *work)
{
    bool ret = false;
    unsigned long flags;
    
    local_irq_save(flags);

    if (!test_and_set_bit(WORK_STRUCT_PENDING_BIT, work_data_bits(work))) {
        __queue_work(cpu, wq, work);
        ret = true;
    }
    
    local_irq_restore(flags);
    return ret;
}

static void __queue_work(int cpu, struct workqueue_struct *wq,
             struct work_struct *work)
{
    insert_work(pwq, work, worklist, work_flags);
}
```

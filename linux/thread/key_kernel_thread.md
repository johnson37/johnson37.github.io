# Key Kernel Thread
- init: this is the first process
- kthreadd: this thread aims to create other kernel thread
- ksoftirqd: this thread aims to handle software irqs.
- kworker: this thread aims to work the bottom of interrupts.
- migration: this thread aims to transfer some work from one CPU to another CPU.
- watchdog: watchdog can reset the system if the system doesn't work fine.
- khelper：这种内核线程只有一个，主要作用是指定用户空间的程序路径和环境变量, 最终运行指定的user space的程序，属于关键线程，不能关闭

## Linux 2.6.14 Thread
### init

#### Create init thread
```c
asmlinkage void __init start_kernel(void)
{
    rest_init();
}
```
```c
static void noinline rest_init(void)
    __releases(kernel_lock)
{
    kernel_thread(init, NULL, CLONE_FS | CLONE_SIGHAND);
    numa_default_policy();
    unlock_kernel();
    preempt_enable_no_resched();

+---  4 lines: The boot idle thread must execute schedule()----------------------------------
    schedule();

    cpu_idle();
} 
```
#### code for init thread
```c
static int init(void * unused)
{
    lock_kernel();
    /*
     * init can run on any cpu.
     */
    set_cpus_allowed(current, CPU_MASK_ALL);
    /*
     * Tell the world that we're going to be the grim
     * reaper of innocent orphaned children.
     *
     * We don't want people to have to make incorrect
     * assumptions about where in the task array this
     * can be found.
     */
    child_reaper = current;
    /* Sets up cpus_possible() */
    smp_prepare_cpus(max_cpus);

    do_pre_smp_initcalls();

    fixup_cpu_present_map();
    smp_init();
    sched_init_smp();

    cpuset_init_smp();

    /*
     * Do this before initcalls, because some drivers want to access
     * firmware files.
     */
    populate_rootfs();

    do_basic_setup();

    /*
     * check if there is an early userspace init.  If yes, let it do all
     * the work
     */
    if (!ramdisk_execute_command)
        ramdisk_execute_command = "/init";

    if (sys_access((const char __user *) ramdisk_execute_command, 0) != 0) {
        ramdisk_execute_command = NULL;
        prepare_namespace();
    }

    /*
     * Ok, we have completed the initial bootup, and
     * we're essentially up and running. Get rid of the
     * initmem segments and start the user-mode stuff..
     */
    free_initmem();
    unlock_kernel();
    system_state = SYSTEM_RUNNING;
    numa_default_policy();

    if (sys_open((const char __user *) "/dev/console", O_RDWR, 0) < 0)
        printk(KERN_WARNING "Warning: unable to open an initial console.\n");

    (void) sys_dup(0);
    (void) sys_dup(0);

    if (ramdisk_execute_command) {
        run_init_process(ramdisk_execute_command);
        printk(KERN_WARNING "Failed to execute %s\n",
                ramdisk_execute_command);
    }

    /*
     * We try each of these until one succeeds.
     *
     * The Bourne shell can be used instead of init if we are 
     * trying to recover a really broken machine.
     */
    if (execute_command) {
        run_init_process(execute_command);
        printk(KERN_WARNING "Failed to execute %s.  Attempting "
                    "defaults...\n", execute_command);
    }
    run_init_process("/sbin/init");
    run_init_process("/etc/init");
    run_init_process("/bin/init");
    run_init_process("/bin/sh");

    panic("No init found.  Try passing init= option to kernel.");
}

```

### ksoftirqd
#### Create ksoftirqd 
```c
static int init(void * unused)
{
    lock_kernel();
    /*
     * init can run on any cpu.
     */
    set_cpus_allowed(current, CPU_MASK_ALL);
    /*
     * Tell the world that we're going to be the grim
     * reaper of innocent orphaned children.
     *
     * We don't want people to have to make incorrect
     * assumptions about where in the task array this
     * can be found.
     */
    child_reaper = current;
    printk(KERN_ERR "Johnson Inin Thread\n");
    /* Sets up cpus_possible() */
    smp_prepare_cpus(max_cpus);

    do_pre_smp_initcalls();
	...
}

static void do_pre_smp_initcalls(void)
{
    extern int spawn_ksoftirqd(void);
#ifdef CONFIG_SMP
    extern int migration_init(void);

    migration_init();
#endif
    spawn_ksoftirqd();
    spawn_softlockup_task();
}

__init int spawn_ksoftirqd(void)
{
    void *cpu = (void *)(long)smp_processor_id();
    cpu_callback(&cpu_nfb, CPU_UP_PREPARE, cpu);
    cpu_callback(&cpu_nfb, CPU_ONLINE, cpu);
    register_cpu_notifier(&cpu_nfb);
    return 0;
}
static int __devinit cpu_callback(struct notifier_block *nfb,
                  unsigned long action,
                  void *hcpu)
{
    int hotcpu = (unsigned long)hcpu;
    struct task_struct *p;

    switch (action) {
    case CPU_UP_PREPARE:
        BUG_ON(per_cpu(tasklet_vec, hotcpu).list);
        BUG_ON(per_cpu(tasklet_hi_vec, hotcpu).list);
        printk(KERN_ERR "Johnson up ksoftirqd\n");
        p = kthread_create(ksoftirqd, hcpu, "ksoftirqd/%d", hotcpu);
        if (IS_ERR(p)) {
            printk("ksoftirqd for %i failed\n", hotcpu);
            return NOTIFY_BAD;
        }
        kthread_bind(p, hotcpu);
        per_cpu(ksoftirqd, hotcpu) = p;
        break;
    case CPU_ONLINE:
        wake_up_process(per_cpu(ksoftirqd, hotcpu));
        break;
#ifdef CONFIG_HOTPLUG_CPU
    case CPU_UP_CANCELED:
        /* Unbind so it can run.  Fall thru. */
        kthread_bind(per_cpu(ksoftirqd, hotcpu), smp_processor_id());
    case CPU_DEAD:
        p = per_cpu(ksoftirqd, hotcpu);
        per_cpu(ksoftirqd, hotcpu) = NULL;
        kthread_stop(p);
        takeover_tasklets(hotcpu);
        break;
#endif /* CONFIG_HOTPLUG_CPU */
    }
    return NOTIFY_OK;
}

```
#### code of ksoftirqd
```c
static int ksoftirqd(void * __bind_cpu)
{
    set_user_nice(current, 19);
    current->flags |= PF_NOFREEZE;

    set_current_state(TASK_INTERRUPTIBLE);

    while (!kthread_should_stop()) {
        preempt_disable();
        if (!local_softirq_pending()) {
            preempt_enable_no_resched();
            schedule();
            preempt_disable();
        }

        __set_current_state(TASK_RUNNING);

        while (local_softirq_pending()) {
            /* Preempt disable stops cpu going offline.
               If already offline, we'll be on wrong CPU:
               don't process */
            if (cpu_is_offline((long)__bind_cpu))
                goto wait_to_die;
            do_softirq();
            preempt_enable_no_resched();
            cond_resched();
            preempt_disable();
        }
        preempt_enable();
        set_current_state(TASK_INTERRUPTIBLE);
    }
    __set_current_state(TASK_RUNNING);
    return 0;
	
wait_to_die:
    preempt_enable();
    /* Wait for kthread_stop */
    set_current_state(TASK_INTERRUPTIBLE);
    while (!kthread_should_stop()) {
        schedule();
        set_current_state(TASK_INTERRUPTIBLE);
    }
    __set_current_state(TASK_RUNNING);
    return 0;
}

```


### events
#### create thread events
```c
static int init(void * unused)
{
    do_pre_smp_initcalls();
	...
	do_basic_setup();
}
```

```c
static void __init do_basic_setup(void)
{
    /* drivers will send hotplug events */
    init_workqueues();
    usermodehelper_init();
    driver_init();

#ifdef CONFIG_SYSCTL
    sysctl_init();
#endif

    /* Networking initialization needs a process context */
    sock_init();

    do_initcalls();
}

```
```c
void init_workqueues(void)
{
    hotcpu_notifier(workqueue_cpu_callback, 0);
    keventd_wq = create_workqueue("events");
    BUG_ON(!keventd_wq);
}
#define create_workqueue(name) __create_workqueue((name), 0)
struct workqueue_struct *__create_workqueue(const char *name,
                        int singlethread)
{
    int cpu, destroy = 0;
    struct workqueue_struct *wq;
    struct task_struct *p;

    wq = kzalloc(sizeof(*wq), GFP_KERNEL);
    if (!wq)
        return NULL;

    wq->name = name;
    /* We don't need the distraction of CPUs appearing and vanishing. */
    lock_cpu_hotplug();
    if (singlethread) {
        INIT_LIST_HEAD(&wq->list);
        p = create_workqueue_thread(wq, 0);
        if (!p)
            destroy = 1;
        else
            wake_up_process(p);
    } else {
        spin_lock(&workqueue_lock);
        list_add(&wq->list, &workqueues);
        spin_unlock(&workqueue_lock);
        for_each_online_cpu(cpu) {
            p = create_workqueue_thread(wq, cpu);
            if (p) {
                kthread_bind(p, cpu);
                wake_up_process(p);
            } else
                destroy = 1;
        }
    }
    unlock_cpu_hotplug();

    /*
     * Was there any error during startup? If yes then clean up:
     */
    if (destroy) {
        destroy_workqueue(wq);
        wq = NULL;
    }   
    return wq;
}

static struct task_struct *create_workqueue_thread(struct workqueue_struct *wq,
                           int cpu)
{
    struct cpu_workqueue_struct *cwq = wq->cpu_wq + cpu;
    struct task_struct *p;

    spin_lock_init(&cwq->lock);
    cwq->wq = wq;
    cwq->thread = NULL;
    cwq->insert_sequence = 0;
    cwq->remove_sequence = 0;
    INIT_LIST_HEAD(&cwq->worklist);
    init_waitqueue_head(&cwq->more_work);
    init_waitqueue_head(&cwq->work_done);
    if (is_single_threaded(wq))
        p = kthread_create(worker_thread, cwq, "%s", wq->name);
    else
        p = kthread_create(worker_thread, cwq, "%s/%d", wq->name, cpu);
    if (IS_ERR(p))
        return NULL;
    cwq->thread = p;
    return p;
}

```
#### code for event thread

```c
static int worker_thread(void *__cwq)
{   
    struct cpu_workqueue_struct *cwq = __cwq;
    DECLARE_WAITQUEUE(wait, current);
    struct k_sigaction sa;
    sigset_t blocked;
    
    current->flags |= PF_NOFREEZE;
    
    set_user_nice(current, -5);

    /* Block and flush all signals */
    sigfillset(&blocked);
    sigprocmask(SIG_BLOCK, &blocked, NULL);
    flush_signals(current);
    
    /* SIG_IGN makes children autoreap: see do_notify_parent(). */
    sa.sa.sa_handler = SIG_IGN;
    sa.sa.sa_flags = 0;
    siginitset(&sa.sa.sa_mask, sigmask(SIGCHLD));
    do_sigaction(SIGCHLD, &sa, (struct k_sigaction *)0);
        
    set_current_state(TASK_INTERRUPTIBLE);
    while (!kthread_should_stop()) {
        add_wait_queue(&cwq->more_work, &wait);
        if (list_empty(&cwq->worklist))
            schedule();
        else
            __set_current_state(TASK_RUNNING);
        remove_wait_queue(&cwq->more_work, &wait);

        if (!list_empty(&cwq->worklist))
            run_workqueue(cwq);
        set_current_state(TASK_INTERRUPTIBLE);
    }
    __set_current_state(TASK_RUNNING);
    return 0;
}

```

### khelper

#### create khelper thread
```c
static void __init do_basic_setup(void)
{
    /* drivers will send hotplug events */
    init_workqueues();
    usermodehelper_init();
}

void __init usermodehelper_init(void)
{
    khelper_wq = create_singlethread_workqueue("khelper");
    BUG_ON(!khelper_wq);
}
#define create_singlethread_workqueue(name) __create_workqueue((name), 1)

```

#### function for khelper
** 这种内核线程只有一个，主要作用是指定用户空间的程序路径和环境变量, 最终运行指定的user space的程序，属于关键线程，不能关闭**
```c
static int worker_thread(void *__cwq)
{   
    struct cpu_workqueue_struct *cwq = __cwq;
    DECLARE_WAITQUEUE(wait, current);
    struct k_sigaction sa;
    sigset_t blocked;
    
    current->flags |= PF_NOFREEZE;
    
    set_user_nice(current, -5);
    
    /* Block and flush all signals */
    sigfillset(&blocked);
    sigprocmask(SIG_BLOCK, &blocked, NULL);
    flush_signals(current);
                        
    /* SIG_IGN makes children autoreap: see do_notify_parent(). */
    sa.sa.sa_handler = SIG_IGN;
    sa.sa.sa_flags = 0;
    siginitset(&sa.sa.sa_mask, sigmask(SIGCHLD));
    do_sigaction(SIGCHLD, &sa, (struct k_sigaction *)0);
    
    set_current_state(TASK_INTERRUPTIBLE);
    while (!kthread_should_stop()) {
        add_wait_queue(&cwq->more_work, &wait);
        if (list_empty(&cwq->worklist))
            schedule();
        else
            __set_current_state(TASK_RUNNING);
        remove_wait_queue(&cwq->more_work, &wait);

        if (!list_empty(&cwq->worklist))
            run_workqueue(cwq);
        set_current_state(TASK_INTERRUPTIBLE);
    }
    __set_current_state(TASK_RUNNING);
    return 0;
}

int call_usermodehelper_keys(char *path, char **argv, char **envp,
                 struct key *session_keyring, int wait)
{
    DECLARE_COMPLETION(done); 
    struct subprocess_info sub_info = {
        .complete   = &done,
        .path       = path,
        .argv       = argv,
        .envp       = envp,
        .ring       = session_keyring,
        .wait       = wait,
        .retval     = 0,
    };
    DECLARE_WORK(work, __call_usermodehelper, &sub_info);
    
    if (!khelper_wq)
        return -EBUSY;
    
    if (path[0] == '\0')
        return 0;
        
    queue_work(khelper_wq, &work);
    wait_for_completion(&done);
    return sub_info.retval;
}  
```

### kthread

####create thread "kthread"
```c
static __init int helper_init(void)
{
    helper_wq = create_singlethread_workqueue("kthread");
    BUG_ON(!helper_wq);
        
    return 0; 
}
core_initcall(helper_init);

```

#### Function for this thread "kthread"
** This thread aims to create thread **
```c
struct task_struct *kthread_create(int (*threadfn)(void *data),
                   void *data,
                   const char namefmt[],
                   ...)
{   
    struct kthread_create_info create;
    DECLARE_WORK(work, keventd_create_kthread, &create);
    
    create.threadfn = threadfn;
    create.data = data;
    init_completion(&create.started);
    init_completion(&create.done);
    
    /*
     * The workqueue needs to start up first:
     */
    if (!helper_wq)
        work.func(work.data);
    else {
        queue_work(helper_wq, &work);
        wait_for_completion(&create.done);
    }
    if (!IS_ERR(create.result)) {
        va_list args;
        va_start(args, namefmt);
        vsnprintf(create.result->comm, sizeof(create.result->comm),
              namefmt, args);
        va_end(args);
    }
    
    return create.result;
}


```

### kblockd

#### create the thread "kblockd" 
```c
static int __init genhd_device_init(void)
{
    bdev_map = kobj_map_init(base_probe, &block_subsys_sem);
    blk_dev_init();
    subsystem_register(&block_subsys);
    return 0;
}   
        
subsys_initcall(genhd_device_init);


int __init blk_dev_init(void)
{
    kblockd_workqueue = create_workqueue("kblockd");
    if (!kblockd_workqueue)
        panic("Failed to create kblockd\n");
    
    request_cachep = kmem_cache_create("blkdev_requests",
            sizeof(struct request), 0, SLAB_PANIC, NULL, NULL);

    requestq_cachep = kmem_cache_create("blkdev_queue",
            sizeof(request_queue_t), 0, SLAB_PANIC, NULL, NULL);

    iocontext_cachep = kmem_cache_create("blkdev_ioc",
            sizeof(struct io_context), 0, SLAB_PANIC, NULL, NULL);
                  
    blk_max_low_pfn = max_low_pfn;
    blk_max_pfn = max_pfn;

    return 0;
}

```
#### Function for kblockd
```c
static void blk_unplug_timeout(unsigned long data)
{
    request_queue_t *q = (request_queue_t *)data;

    kblockd_schedule_work(&q->unplug_work);
} 
```

```c
int kblockd_schedule_work(struct work_struct *work)
{
    return queue_work(kblockd_workqueue, work);
}

EXPORT_SYMBOL(kblockd_schedule_work);

```

### pdflush
#### Create one thread "pdflush"
```c
static int __init pdflush_init(void)
{
    int i;
    for (i = 0; i < MIN_PDFLUSH_THREADS; i++)
        start_one_pdflush_thread();
    return 0;
}

module_init(pdflush_init);


static void start_one_pdflush_thread(void)
{
    kthread_run(pdflush, NULL, "pdflush");
}
```

#### Function for pdflush
**pdflush aims to flush the modify page to HardDisk**
```c
static void sysrq_handle_sync(int key, struct pt_regs *pt_regs,
                  struct tty_struct *tty)
{
    emergency_sync();
}

void emergency_sync(void)
{
    pdflush_operation(do_sync, 0);
}

int pdflush_operation(void (*fn)(unsigned long), unsigned long arg0)
{
    unsigned long flags;      
    int ret = 0;

    if (fn == NULL)
        BUG();      /* Hard to diagnose if it's deferred */

    spin_lock_irqsave(&pdflush_lock, flags);
    if (list_empty(&pdflush_list)) {
        spin_unlock_irqrestore(&pdflush_lock, flags);
        ret = -1;
    } else {
        struct pdflush_work *pdf;      

        pdf = list_entry(pdflush_list.next, struct pdflush_work, list);
        list_del_init(&pdf->list);     
        if (list_empty(&pdflush_list)) 
            last_empty_jifs = jiffies;     
        pdf->fn = fn;
        pdf->arg0 = arg0;
        wake_up_process(pdf->who);
        spin_unlock_irqrestore(&pdflush_lock, flags);
    }
    return ret;
}

```

### aio
#### create one thread for aio
```c
static int __init aio_setup(void)
{
    kiocb_cachep = kmem_cache_create("kiocb", sizeof(struct kiocb),
                0, SLAB_HWCACHE_ALIGN|SLAB_PANIC, NULL, NULL);
    kioctx_cachep = kmem_cache_create("kioctx", sizeof(struct kioctx),
                0, SLAB_HWCACHE_ALIGN|SLAB_PANIC, NULL, NULL);
    printk(KERN_ERR "Create aio\n");
    aio_wq = create_workqueue("aio");

    pr_debug("aio_setup: sizeof(struct page) = %d\n", (int)sizeof(struct page));

    return 0;
}
	
```
#### Function for aio
```c
static DECLARE_WORK(fput_work, aio_fput_routine, NULL);

static void aio_fput_routine(void *data)
{
    spin_lock_irq(&fput_lock);
    while (likely(!list_empty(&fput_head))) {
        struct kiocb *req = list_kiocb(fput_head.next);
        struct kioctx *ctx = req->ki_ctx;

        list_del(&req->ki_list);
        spin_unlock_irq(&fput_lock);

        /* Complete the fput */
        __fput(req->ki_filp);

        /* Link the iocb into the context's free list */
        spin_lock_irq(&ctx->ctx_lock);
        really_put_req(ctx, req);
        spin_unlock_irq(&ctx->ctx_lock);

        put_ioctx(ctx);
        spin_lock_irq(&fput_lock);
    }
    spin_unlock_irq(&fput_lock);
}

static int __aio_put_req(struct kioctx *ctx, struct kiocb *req)
{
    dprintk(KERN_DEBUG "aio_put(%p): f_count=%d\n",
        req, atomic_read(&req->ki_filp->f_count));

    req->ki_users --;
    if (unlikely(req->ki_users < 0))
        BUG();
    if (likely(req->ki_users))
        return 0;
    list_del(&req->ki_list);        /* remove from active_reqs */
    req->ki_cancel = NULL;
    req->ki_retry = NULL;

    /* Must be done under the lock to serialise against cancellation.
     * Call this aio_fput as it duplicates fput via the fput_work.
     */
    if (unlikely(rcuref_dec_and_test(&req->ki_filp->f_count))) {
        get_ioctx(ctx);
        spin_lock(&fput_lock);
        list_add(&req->ki_list, &fput_head);
        spin_unlock(&fput_lock);
        queue_work(aio_wq, &fput_work);
    } else
        really_put_req(ctx, req);
    return 1;
}

```


### kswapd0

#### Create one thread "kswapd0"
```c
static int __init kswapd_init(void)
{       
    pg_data_t *pgdat;
    swap_setup();
    for_each_pgdat(pgdat)
        pgdat->kswapd
        = find_task_by_pid(kernel_thread(kswapd, pgdat, CLONE_KERNEL));
    total_memory = nr_free_pagecache_pages();
    hotcpu_notifier(cpu_callback, 0);
    return 0;
}

module_init(kswapd_init)

```

```c
static int kswapd(void *p)
{
    unsigned long order;
    pg_data_t *pgdat = (pg_data_t*)p;
    struct task_struct *tsk = current;
    DEFINE_WAIT(wait);
    struct reclaim_state reclaim_state = {
        .reclaimed_slab = 0,
    };
    cpumask_t cpumask;

    daemonize("kswapd%d", pgdat->node_id);
    cpumask = node_to_cpumask(pgdat->node_id);
    if (!cpus_empty(cpumask))
        set_cpus_allowed(tsk, cpumask);
    current->reclaim_state = &reclaim_state;

	    tsk->flags |= PF_MEMALLOC|PF_KSWAPD;

    order = 0;
    for ( ; ; ) {
        unsigned long new_order;

        try_to_freeze();

        prepare_to_wait(&pgdat->kswapd_wait, &wait, TASK_INTERRUPTIBLE);
        new_order = pgdat->kswapd_max_order;
        pgdat->kswapd_max_order = 0;
        if (order < new_order) {
            /*
             * Don't sleep if someone wants a larger 'order'
             * allocation
             */
            order = new_order;
        } else {
            schedule();
            order = pgdat->kswapd_max_order;
        }
        finish_wait(&pgdat->kswapd_wait, &wait);

        balance_pgdat(pgdat, 0, order);
    }
    return 0;
}

```


### kseriod

#### Create one thread "kseriod"
```c
static int __init serio_init(void)
{
    serio_task = kthread_run(serio_thread, NULL, "kseriod");
    if (IS_ERR(serio_task)) {
        printk(KERN_ERR "serio: Failed to start kseriod\n");
        return PTR_ERR(serio_task);
    }   

    serio_bus.dev_attrs = serio_device_attrs;
    serio_bus.drv_attrs = serio_driver_attrs;
    serio_bus.match = serio_bus_match;
    serio_bus.hotplug = serio_hotplug;
    serio_bus.resume = serio_resume; 
    bus_register(&serio_bus);

    return 0;
}
```
####

### mtdblockd
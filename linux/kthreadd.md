# Kthreadd

## Basic Introduction

## Code Flow
```c
//In this period, the function is kthreadd the main function, it will take the responsibility to create kernel threads.
int kthreadd(void *unused)
{
    struct task_struct *tsk = current;

    /* Setup a clean context for our children to inherit. */
    set_task_comm(tsk, "kthreadd");
    ignore_signals(tsk);
    set_cpus_allowed_ptr(tsk, cpu_all_mask);
    set_mems_allowed(node_states[N_MEMORY]);

    current->flags |= PF_NOFREEZE;
    cgroup_init_kthreadd();
    
    for (;;) {
        set_current_state(TASK_INTERRUPTIBLE);
        if (list_empty(&kthread_create_list))
            schedule();
        __set_current_state(TASK_RUNNING);
    
        spin_lock(&kthread_create_lock);
        while (!list_empty(&kthread_create_list)) {
            struct kthread_create_info *create;

            create = list_entry(kthread_create_list.next,
                        struct kthread_create_info, list);
            list_del_init(&create->list);
            spin_unlock(&kthread_create_lock);
        
            create_kthread(create);
            
            spin_lock(&kthread_create_lock);
        }
        spin_unlock(&kthread_create_lock);
    }
        
    return 0;
}

```

```c
#define kthread_create(threadfn, data, namefmt, arg...) \
    kthread_create_on_node(threadfn, data, NUMA_NO_NODE, namefmt, ##arg)

struct task_struct *kthread_create_on_node(int (*threadfn)(void *data),
                       void *data, int node,
                       const char namefmt[],
                       ...)
{
    DECLARE_COMPLETION_ONSTACK(done);
    struct task_struct *task;
    struct kthread_create_info *create = kmalloc(sizeof(*create),
                             GFP_KERNEL);

    if (!create)
        return ERR_PTR(-ENOMEM);
    create->threadfn = threadfn;
    create->data = data;
    create->node = node;
    create->done = &done;

    spin_lock(&kthread_create_lock);
    list_add_tail(&create->list, &kthread_create_list);
    spin_unlock(&kthread_create_lock);

    wake_up_process(kthreadd_task);
    /*
     * Wait for completion in killable state, for I might be chosen by
     * the OOM killer while kthreadd is trying to allocate memory for
     * new kernel thread.
     */
    if (unlikely(wait_for_completion_killable(&done))) {
        /*
         * If I was SIGKILLed before kthreadd (or new kernel thread)
         * calls complete(), leave the cleanup of this structure to
         * that thread.
         */
        if (xchg(&create->done, NULL))
            return ERR_PTR(-EINTR);
        /*
         * kthreadd (or new kernel thread) will call complete()
         * shortly.
         */
        wait_for_completion(&done);
    }
    task = create->result;
    if (!IS_ERR(task)) {
        static const struct sched_param param = { .sched_priority = 0 };
        char name[TASK_COMM_LEN];
        va_list args;

        va_start(args, namefmt);
        /*
         * task is already visible to other tasks, so updating
         * COMM must be protected.
         */
        vsnprintf(name, sizeof(name), namefmt, args);
        set_task_comm(task, name);
        va_end(args);
        /*
         * root may have changed our (kthreadd's) priority or CPU mask.
         * The kernel thread should not inherit these properties.
         */
        sched_setscheduler_nocheck(task, SCHED_NORMAL, &param);
        set_cpus_allowed_ptr(task, cpu_all_mask);
    }
    kfree(create);
    return task;
}

```
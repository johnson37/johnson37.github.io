# Linux startup

## Linux Version 2.6.14
```c
asmlinkage void __init start_kernel(void)
{
	...
	rest_init();
}

// Here, we create init thread.

static void noinline rest_init(void)
    __releases(kernel_lock)
{
    kernel_thread(init, NULL, CLONE_FS | CLONE_SIGHAND);
    numa_default_policy();
    unlock_kernel();
    preempt_enable_no_resched();

    /*
     * The boot idle thread must execute schedule()
     * at least one to get things moving:
     */
    schedule();

    cpu_idle();
} 

static int init(void * unused)
{
	...
    do_basic_setup();
}

static void __init do_basic_setup(void)
{
    /* drivers will send hotplug events */
    printk(KERN_ERR "Johnson pre create events thread\n");
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


//Here, we will load lots of kernel modules. module_init..
static void __init do_initcalls(void)
{
    initcall_t *call;
    int count = preempt_count();
    printk(KERN_ERR "Johnson start do_initcalls\n");
    for (call = __initcall_start; call < __initcall_end; call++) {
        char *msg;

        //if (initcall_debug) {
        {
            printk(KERN_ERR "Calling initcall 0x%p", *call);
            print_fn_descriptor_symbol(": %s()", (unsigned long) *call);
            printk("\n");
        }

        (*call)();

        msg = NULL;
        if (preempt_count() != count) {
            msg = "preemption imbalance";
            preempt_count() = count;
        }
        if (irqs_disabled()) {
            msg = "disabled interrupts";
            local_irq_enable();
        }
        if (msg) {
            printk(KERN_WARNING "error in initcall at 0x%p: "
                "returned with %s\n", *call, msg);
        }
    }

    /* Make sure there is no pending stuff from the initcall sequence */
    flush_scheduled_work();
}

```


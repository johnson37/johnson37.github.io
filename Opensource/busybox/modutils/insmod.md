# insmod

insmod的用处是加载一个ko模块。通过系统调用的方式来实现。

## Application Layer

```c
int insmod_main(int argc UNUSED_PARAM, char **argv)
{
    rc = bb_init_module(filename, parse_cmdline_module_options(argv));
}

int FAST_FUNC bb_init_module(const char *filename, const char *options)
{
    init_module(image, image_size, options);
}

# define init_module(mod, len, opts) syscall(__NR_init_module, mod, len, opts)
```

## Kernel Layer
```c
SYSCALL_DEFINE3(init_module, void __user *, umod,
        unsigned long, len, const char __user *, uargs)
{
    int err;
    struct load_info info = { };

    err = may_init_module();
    if (err)
        return err;

    pr_debug("init_module: umod=%p, len=%lu, uargs=%p\n",
           umod, len, uargs);

    err = copy_module_from_user(umod, len, &info);
    if (err)
        return err;

    return load_module(&info, uargs, 0);
}

static int load_module(struct load_info *info, const char __user *uargs,
               int flags)
{
    return do_init_module(mod);
}

static noinline int do_init_module(struct module *mod)
{
    if (mod->init != NULL)
        ret = do_one_initcall(mod->init);

}

int __init_or_module do_one_initcall(initcall_t fn)
{
    int count = preempt_count();
    int ret;
    char msgbuf[64];

    if (initcall_blacklisted(fn))
        return -EPERM;

    if (initcall_debug)
        ret = do_one_initcall_debug(fn);
    else
        ret = fn();

    msgbuf[0] = 0;

    if (preempt_count() != count) {
        sprintf(msgbuf, "preemption imbalance ");
        preempt_count_set(count);
    }
    if (irqs_disabled()) {
        strlcat(msgbuf, "disabled interrupts ", sizeof(msgbuf));
        local_irq_enable();
    }
    WARN(msgbuf[0], "initcall %pF returned with %s\n", fn, msgbuf);

    return ret;
}

```

# rmmod

rmmod 的作用是卸载对应的module

## Application flow
```c
int rmmod_main(int argc UNUSED_PARAM, char **argv)
{
        if (bb_delete_module(modname, flags))
            bb_error_msg_and_die("can't unload '%s': %s",
                         modname, moderror(errno));

}
int FAST_FUNC bb_delete_module(const char *module, unsigned int flags)
{
    errno = 0;
    delete_module(module, flags);
    return errno;
}

# define delete_module(mod, flags) syscall(__NR_delete_module, mod, flags)
```

## Kernel part
```c
SYSCALL_DEFINE2(delete_module, const char __user *, name_user,
        unsigned int, flags)
{
    if (mod->exit != NULL)
        mod->exit();
}
```
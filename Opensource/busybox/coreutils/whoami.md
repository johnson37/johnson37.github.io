# whoami

whoami 命令的目的是获得当前进程下的有效用户名。因为Linux是个多用户系统，当前系统内的不同进程可能是不同的用户起的。
所以在不同的进程中执行whoami的结果有可能是不同的。

## code flow
whoami 是busybox中一个命令，调用api geteuid来获取对应的uid
```c
int whoami_main(int argc, char **argv) MAIN_EXTERNALLY_VISIBLE;
int whoami_main(int argc UNUSED_PARAM, char **argv UNUSED_PARAM)
{
    if (argv[1])
        bb_show_usage();

    /* Will complain and die if username not found */
    puts(xuid2uname(geteuid()));

    return fflush_all();
}

```

geteuid 是libc中的一个api，他会进一步调用系统调用geteuid

```c
SYSCALL_DEFINE0(geteuid)
{
    /* Only we change this so SMP safe */
    return from_kuid_munged(current_user_ns(), current_euid());
}

#define current_euid()      (current_cred_xxx(euid))

#define current_cred_xxx(xxx)           \
({                      \
    current_cred()->xxx;            \
})

#define current_cred() \
    rcu_dereference_protected(current->cred, 1)

```

current是一个进程结构体，current里面包含了cred结构体，里面有euid数据。
curent就是当前做系统调用的进程对应的PCB current。
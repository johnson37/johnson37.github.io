# init

init 进程是系统起的第一个用户态的进程。
Linux系统首先由uboot启动，uboot进行基本的初始化之后，会带Kernel，在start_kernel当中，会调用init应用。

## Code Flow 
### How init process is triggered.

```c
asmlinkage __visible void __init start_kernel(void)
{
    rest_init();
}

static noinline void __init_refok rest_init(void)
{
    kernel_thread(kernel_init, NULL, CLONE_FS);
}

static int __ref kernel_init(void *unused)
{
    if (!try_to_run_init_process("/sbin/init") ||
        !try_to_run_init_process("/etc/init") ||
        !try_to_run_init_process("/bin/init") ||
        !try_to_run_init_process("/bin/sh"))
        return 0;
}

static int run_init_process(const char *init_filename)
{
    argv_init[0] = init_filename;
    return do_execve(getname_kernel(init_filename),
        (const char __user *const __user *)argv_init,
        (const char __user *const __user *)envp_init);
}

```

###  init process

```c
int init_main(int argc UNUSED_PARAM, char **argv)
{
    if (argv[1]
     && (strcmp(argv[1], "single") == 0 || strcmp(argv[1], "-s") == 0 || LONE_CHAR(argv[1], '1'))
    ) {
        /* ??? shouldn't we set RUNLEVEL="b" here? */
        /* Start a shell on console */
        new_init_action(RESPAWN, bb_default_login_shell, "");
    } else {
        /* Not in single user mode - see what inittab says */

        /* NOTE that if CONFIG_FEATURE_USE_INITTAB is NOT defined,
         * then parse_inittab() simply adds in some default
         * actions(i.e., INIT_SCRIPT and a pair
         * of "askfirst" shells */
        parse_inittab();
    }
	
}

static void parse_inittab(void)
{

    char *token[4];
    parser_t *parser = config_open2("/etc/inittab", fopen_for_read);

	#if ENABLE_FEATURE_USE_INITTAB
    /* optional_tty:ignored_runlevel:action:command
     * Delims are not to be collapsed and need exactly 4 tokens
     */
    while (config_read(parser, token, 4, 0, "#:",
                PARSE_NORMAL & ~(PARSE_TRIM | PARSE_COLLAPSE))) {
        /* order must correspond to SYSINIT..RESTART constants */
        static const char actions[] ALIGN1 =
            "sysinit\0""wait\0""once\0""respawn\0""askfirst\0"
            "ctrlaltdel\0""shutdown\0""restart\0";
        int action;
        char *tty = token[0];

        if (!token[3]) /* less than 4 tokens */
            goto bad_entry;
        action = index_in_strings(actions, token[2]);
        if (action < 0 || !token[3][0]) /* token[3]: command */
            goto bad_entry;
        /* turn .*TTY -> /dev/TTY */
        if (tty[0]) {
            if (strncmp(tty, "/dev/", 5) == 0)
                tty += 5;
            tty = concat_path_file("/dev/", tty);
        }
        new_init_action(1 << action, token[3], tty);
        if (tty[0])
            free(tty);
        continue;
 bad_entry:
        message(L_LOG | L_CONSOLE, "Bad inittab entry at line %d",
                parser->lineno);
    }
    config_close(parser);
#endif
}


```

```c
cat inittab 
#
#       $Id: inittab,v 1.2 2006/04/27 21:40:03 jwessel Exp $
#
# Busybox's init uses a different syntax then the normal init.
# The "-" in -/bin/sh allows job control on /dev/console.
#
::sysinit:/etc/init.d/rcS
```

在rcS可执行脚本可以继续执行启动脚本。

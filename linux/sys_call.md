# Sys call
In Linux system, the system call is implemented by software interrupt.

```c
int open(const char * filename, int flag, ...)
{
    register int res;
    va_list arg;

    va_start(arg, flag);
    __asm__("movl %2,%%ebx\n\t"
            "int $0x80"
            :"=a"(res)
            :"0"(__NR_open), "g"((long)(filename)), "c"(flag),
            "d"(va_arg(arg, int)));
    if (res >= 0)
        return res;
    errno = -res;
    return -1; 
}

```

```c
asmlinkage void start_kernel(void)
{
	sched_init();
}

void sched_init(void)
{
	set_system_gate(0x80, &system_call);
}

```

```x86asm
_system_call:
	...
	...
	call _sys_call_table(,%eax,4)
	...
	jmp ret_from_sys_call

ret_from_sys_call:
	...
	jne signal_return
	
signal_return:
	...
	call _do_signal
```
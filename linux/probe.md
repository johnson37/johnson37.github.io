# Probe Debug in Linux Kernel

## Basic Introduction
Linux 提供了一种调试内核的手段。在Linux源码中sample有具体的例子。
## probe types
probe分成kprobe，jprobe以及retprobe
- kprobe: kprobe是基础，可以打印trigger某个函数之前的一些寄存器的信息。当然可以查看Linux内核全局变量在此处的值。
- jprobe: jprobe主要用于打印函数入参
- retprobe: retprobe 主要用于函数返回值的打印
### Kprobe
```c
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>
#include <linux/proc_fs.h>
#include <linux/seq_file.h>

#define MAX_WP 10
char * probe_funs[]={"ip_rcv","ip_local_deliver"};
struct Netprobe
{
    struct kprobe kp;
    unsigned int count;
};

typedef struct Netprobe probe;

probe myprobe[MAX_WP];

void probe_init(void)
{
    int loop = 0;
    int count = sizeof(probe_funs)/4;
    printk("Size is %d\n", sizeof(probe_funs));
    while (loop < count)
    {
        myprobe[loop].kp.symbol_name = probe_funs[loop];
        myprobe[loop].count = 0;
        loop++;
    }
}

static int handler_pre(struct kprobe *p, struct pt_regs *regs)
{
    struct Netprobe * temp_probe = (struct Netprobe *)p;
    //ip_rcv_count++;
    temp_probe->count = temp_probe->count + 1;
    return 0;
}

//Add Proc file
static struct proc_dir_entry *proc_dir, *proc_file; 
static int sfunet_proc_show(struct seq_file *m, void *v)
{
    int loop = 0;
    int count = sizeof(probe_funs)/4;
    printk("Size is %d\n", sizeof(probe_funs));
    while (loop < count)
    {   
        seq_printf(m, "Watch Point %s ,count %d\n", myprobe[loop].kp.symbol_name, myprobe[loop].count);
        loop++;
    }   

    return 0;
}
static int sfunet_proc_open(struct inode * inode, struct file * file)
{
    return single_open(file, sfunet_proc_show, NULL);
}
static struct file_operations sfunet_fops =
{
    .owner = THIS_MODULE,
    .open = sfunet_proc_open,
    .release = single_release,
    .read = seq_read,
    .llseek = seq_lseek,
};
static int kprobe_proc_init(void)
{
    proc_dir = proc_mkdir("kprobe_example", NULL);
    if (!proc_dir)
    {
        printk(KERN_ERR "Proc dir failed\n");
        return -ENOMEM;
    }
    proc_file = proc_create("count_info", 0644, proc_dir, &sfunet_fops);
    if (!proc_file)
    {   
        return -ENOMEM;
    }
    printk(KERN_ERR "Proc file success\n");
    return 0;
}

static int kprobe_proc_exit(void)
{
    remove_proc_entry("count_info", proc_dir);
    remove_proc_entry("kprobe_example", NULL);
    return 0;
}

static int __init kprobe_init(void)
{ 
    int ret; 
    int count = sizeof(probe_funs)/4;
    int loop = 0;
    probe_init();
    while (loop < count)
    {
        myprobe[loop].kp.symbol_name = probe_funs[loop];
        myprobe[loop].kp.pre_handler = handler_pre;
        myprobe[loop].kp.post_handler = NULL;
        myprobe[loop].kp.fault_handler = NULL;  
        ret = register_kprobe(&(myprobe[loop].kp)); 
        if (ret < 0) 
        { 
            printk(KERN_INFO "register_kprobe failed, returned %d\n", ret); 
            return ret; 
        } 
        loop++;
    }
    kprobe_proc_init();
    return 0; 
} 
static void __exit kprobe_exit(void)
{ 
    int count = sizeof(probe_funs)/4;
    int loop = 0;
    while ( loop < count)
    {
        unregister_kprobe(&(myprobe[loop].kp));
        loop++;
    }
    kprobe_proc_exit();
} 
module_init(kprobe_init) 
module_exit(kprobe_exit) 
MODULE_LICENSE("GPL");

```
### Jprobe

```c
/*
 * Here's a sample kernel module showing the use of jprobes to dump
 * the arguments of do_fork().
 *
 * For more information on theory of operation of jprobes, see
 * Documentation/kprobes.txt
 *
 * Build and insert the kernel module as done in the kprobe example.
 * You will see the trace data in /var/log/messages and on the
 * console whenever do_fork() is invoked to create a new process.
 * (Some messages may be suppressed if syslogd is configured to
 * eliminate duplicate messages.)
 */

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>
#include <linux/skbuff.h>
#include <linux/netdevice.h>
/*
 * Jumper probe for do_fork.
 * Mirror principle enables access to arguments of the probed routine
 * from the probe handler.
 */

/* Proxy routine having the same arguments as actual do_fork() routine */
static long jip_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt, struct net_device *orig_dev)
{
	printk(KERN_ERR "jprobe: dev = %s, orig_dev = %s, packet type = %d", dev->name ,orig_dev->name, skb->pkt_type);

	/* Always end with a call to jprobe_return(). */
	jprobe_return();
	return 0;
}

static struct jprobe my_jprobe = {
	.entry			= jip_rcv,
	.kp = {
		.symbol_name	= "ip_rcv",
	},
};

static int __init jprobe_init(void)
{
	int ret;

	ret = register_jprobe(&my_jprobe);
	if (ret < 0) {
		printk(KERN_INFO "register_jprobe failed, returned %d\n", ret);
		return -1;
	}
	printk(KERN_INFO "Planted jprobe at %p, handler addr %p\n",
	       my_jprobe.kp.addr, my_jprobe.entry);
	return 0;
}

static void __exit jprobe_exit(void)
{
	unregister_jprobe(&my_jprobe);
	printk(KERN_INFO "jprobe at %p unregistered\n", my_jprobe.kp.addr);
}

module_init(jprobe_init)
module_exit(jprobe_exit)
MODULE_LICENSE("GPL");

```
### RetProbe
```c
/*
 * kretprobe_example.c
 *
 * Here's a sample kernel module showing the use of return probes to
 * report the return value and total time taken for probed function
 * to run.
 *
 * usage: insmod kretprobe_example.ko func=<func_name>
 *
 * If no func_name is specified, do_fork is instrumented
 *
 * For more information on theory of operation of kretprobes, see
 * Documentation/kprobes.txt
 *
 * Build and insert the kernel module as done in the kprobe example.
 * You will see the trace data in /var/log/messages and on the console
 * whenever the probed function returns. (Some messages may be suppressed
 * if syslogd is configured to eliminate duplicate messages.)
 */

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>
#include <linux/ktime.h>
#include <linux/limits.h>
#include <linux/sched.h>

static char func_name[NAME_MAX] = "do_fork";
module_param_string(func, func_name, NAME_MAX, S_IRUGO);
MODULE_PARM_DESC(func, "Function to kretprobe; this module will report the"
			" function's execution time");

/*
 * Return-probe handler: Log the return value and duration. Duration may turn
 * out to be zero consistently, depending upon the granularity of time
 * accounting on the platform.
 */
static int ret_handler(struct kretprobe_instance *ri, struct pt_regs *regs)
{
	int retval = regs_return_value(regs);
	printk(KERN_INFO "%s returned %d \n",
			func_name, retval);
	return 0;
}

static struct kretprobe my_kretprobe = {
	.handler		= ret_handler,
	.entry_handler		= NULL,
	/* Probe up to 20 instances concurrently. */
	.maxactive		= 20,
};

static int __init kretprobe_init(void)
{
	int ret;

	my_kretprobe.kp.symbol_name = func_name;
	ret = register_kretprobe(&my_kretprobe);
	if (ret < 0) {
		printk(KERN_INFO "register_kretprobe failed, returned %d\n",
				ret);
		return -1;
	}
	printk(KERN_INFO "Planted return probe at %s: %p\n",
			my_kretprobe.kp.symbol_name, my_kretprobe.kp.addr);
	return 0;
}

static void __exit kretprobe_exit(void)
{
	unregister_kretprobe(&my_kretprobe);
	printk(KERN_INFO "kretprobe at %p unregistered\n",
			my_kretprobe.kp.addr);

	/* nmissed > 0 suggests that maxactive was set too low. */
	printk(KERN_INFO "Missed probing %d instances of %s\n",
		my_kretprobe.nmissed, my_kretprobe.kp.symbol_name);
}

module_init(kretprobe_init)
module_exit(kretprobe_exit)
MODULE_LICENSE("GPL");

```
# drop_cache

## How to check system's memory
```c
sync; echo 3 > /proc/sys/vm/drop_caches; free
```
To use /proc/sys/vm/drop_caches, just echo a number to it.

To free pagecache:
```
# echo 1 > /proc/sys/vm/drop_caches
```
To free dentries and inodes:
```
# echo 2 > /proc/sys/vm/drop_caches
```
To free pagecache, dentries and inodes:
```
echo 3 > /proc/sys/vm/drop_caches
```
This is a non-destructive operation and will only free things 
that are completely unused. Dirty objects will continue to be 
in use until written out to disk and are not freeable. 
If you run "sync" first to flush them out to disk, these drop operations will tend to free more memory. 

## Code Flow
```c
static ssize_t proc_sys_call_handler(struct file *filp, void __user *buf,
        size_t count, loff_t *ppos, int write)
{
    struct inode *inode = file_inode(filp);
    struct ctl_table_header *head = grab_header(inode);
    struct ctl_table *table = PROC_I(inode)->sysctl_entry;
    ssize_t error;
    size_t res;

    if (IS_ERR(head))
        return PTR_ERR(head);

+---  4 lines: At this point we know that the sysctl was not unregistered------------------------------------------------------------------------------------------------
    error = -EPERM;
    if (sysctl_perm(head, table, write ? MAY_WRITE : MAY_READ))
        goto out;

    /* if that can happen at all, it should be -EINVAL, not -EISDIR */
    error = -EINVAL;
    if (!table->proc_handler)
        goto out;

    /* careful: calling conventions are nasty here */
    res = count;
    error = table->proc_handler(table, write, buf, &res, ppos);
    if (!error)
        error = res;
out:
    sysctl_head_finish(head);

    return error;
}

```

```c
static struct ctl_table vm_table[] = {
    {
        .procname   = "drop_caches",
        .data       = &sysctl_drop_caches,
        .maxlen     = sizeof(int),
        .mode       = 0644,
        .proc_handler   = drop_caches_sysctl_handler,
        .extra1     = &one,
        .extra2     = &four,
    },
}
```

```c
int drop_caches_sysctl_handler(struct ctl_table *table, int write,
    void __user *buffer, size_t *length, loff_t *ppos)
{
    int ret;

    ret = proc_dointvec_minmax(table, write, buffer, length, ppos);
    if (ret)
        return ret;
    if (write) {
        static int stfu;

        if (sysctl_drop_caches & 1) {
            iterate_supers(drop_pagecache_sb, NULL);
            count_vm_event(DROP_PAGECACHE);
        }
        if (sysctl_drop_caches & 2) {
            drop_slab();
            count_vm_event(DROP_SLAB);
        }
        if (!stfu) {
            pr_info("%s (%d): drop_caches: %d\n",
                current->comm, task_pid_nr(current),
                sysctl_drop_caches);
        }
        stfu |= sysctl_drop_caches & 4;
    }
    return 0;
}

```
# Fasync

## Application Layer
- lp_fd = open(KLPD_DEV, O_RDWR);
- signal(SIGIO, io_handler);
- fcntl(lp_fd, F_SETOWN, getpid());
- fcntl(lp_fd, F_SETFL, old_flags | FASYNC);

## Kernel Layer
**Set fasync_helper, so we need which application we send the signal**
```c
static struct file_operations temp_fops = {
.fasync  = TEMP_Fasync,
}

static int TEMP_Fasync(int fd, struct file *file, int mode)
{       
    return fasync_helper(fd, file, mode, &async_queue);
}


```

**Send SIGNAL**
```c
    if (async_queue)
    {
        kill_fasync(&async_queue, SIGIO, POLLIN);
    }
```
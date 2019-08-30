# Fasync

## Application Layer
```c
    temp_fd = open(TEMP_DEV, O_RDWR);
    if (temp_fd < 0)
    { 
        fprintf(stderr, "Cann't open file \n");
        return -1;
    }

    signal(SIGIO, io_handler);
    fcntl(temp_fd, F_SETOWN, getpid());
    old_flags = fcntl(temp_fd, F_GETFL);
    fcntl(temp_fd, F_SETFL, old_flags | FASYNC);
```
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
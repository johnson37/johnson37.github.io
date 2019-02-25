# Linux1.0 Net

## Main Flow
### Send Main Flow (Take UDP as one example)
```c

```
### Receive Main Flow (Take UDP as one example)

## Socket ioctl (Take Route as one example)
In busybox 1.6, route is implemented as below:
```c
	skfd = xsocket(AF_INET, SOCK_DGRAM, 0);

	if (action == RTACTION_ADD)
		xioctl(skfd, SIOCADDRT, &rt);
	else
		xioctl(skfd, SIOCDELRT, &rt);
```

In Linux 1.0 Kernel, socket & ioctl part is as below:

In sock_socket, we will create one new socket structure, one new inode, and one f_inode.
In this f_inode, we will make use of **socket_file_ops**

```c
static int
sock_socket(int family, int type, int protocol)
{
...
  if (!(sock = sock_alloc(1))) {
	printk("sock_socket: no more sockets\n");
	return(-EAGAIN);
  }
  sock->type = type;
  sock->ops = ops;
  if ((i = sock->ops->create(sock, protocol)) < 0) {
	sock_release(sock);
	return(i);
  }

  if ((fd = get_fd(SOCK_INODE(sock))) < 0) {
	sock_release(sock);
	return(-EINVAL);
  }
}

static int
get_fd(struct inode *inode)
{
  int fd;
  struct file *file;

  /* Find a file descriptor suitable for return to the user. */
  file = get_empty_filp();
  if (!file) return(-1);
  for (fd = 0; fd < NR_OPEN; ++fd)
	if (!current->filp[fd]) break;
  if (fd == NR_OPEN) {
	file->f_count = 0;
	return(-1);
  }
  FD_CLR(fd, &current->close_on_exec);
  current->filp[fd] = file;
  file->f_op = &socket_file_ops;
  file->f_mode = 3;
  file->f_flags = 0;
  file->f_count = 1;
  file->f_inode = inode;
  if (inode) inode->i_count++;
  file->f_pos = 0;
  return(fd);
}
static struct file_operations socket_file_ops = {
  sock_lseek,
  sock_read,
  sock_write,
  sock_readdir,
  sock_select,
  sock_ioctl,
  NULL,			/* mmap */
  NULL,			/* no special open code... */
  sock_close
};


```

In ioctl, we call the implementation function through fd's f_op.
```c
asmlinkage int sys_ioctl(unsigned int fd, unsigned int cmd, unsigned long arg)
{
...
...
		default:
			if (filp->f_inode && S_ISREG(filp->f_inode->i_mode))
				return file_ioctl(filp,cmd,arg);

			if (filp->f_op && filp->f_op->ioctl)
				return filp->f_op->ioctl(filp->f_inode, filp, cmd,arg);

			return -EINVAL;	
}


int
sock_ioctl(struct inode *inode, struct file *file, unsigned int cmd,
	   unsigned long arg)
{
  struct socket *sock;

  DPRINTF((net_debug, "NET: sock_ioctl: inode=0x%x cmd=0x%x arg=%d\n",
							inode, cmd, arg));
  if (!(sock = socki_lookup(inode))) {
	printk("NET: sock_ioctl: can't find socket for inode!\n");
	return(-EBADF);
  }
  return(sock->ops->ioctl(sock, cmd, arg));
}

static int
inet_ioctl(struct socket *sock, unsigned int cmd, unsigned long arg)
{
  struct sock *sk;
  int err;

  DPRINTF((DBG_INET, "INET: in inet_ioctl\n"));
  sk = NULL;
  if (sock && (sk = (struct sock *) sock->data) == NULL) {
	printk("AF_INET: Warning: sock->data = NULL: %d\n" , __LINE__);
	return(0);
  }

  switch(cmd) {
	case FIOSETOWN:
	case SIOCSPGRP:
		err=verify_area(VERIFY_READ,(int *)arg,sizeof(long));
		if(err)
			return err;
		if (sk)
			sk->proc = get_fs_long((int *) arg);
		return(0);
	case FIOGETOWN:
	case SIOCGPGRP:
		if (sk) {
			err=verify_area(VERIFY_WRITE,(void *) arg, sizeof(long));
			if(err)
				return err;
			put_fs_long(sk->proc,(int *)arg);
		}
		return(0);
#if 0	/* FIXME: */
	case SIOCATMARK:
		printk("AF_INET: ioctl(SIOCATMARK, 0x%08X)\n",(void *) arg);
		return(-EINVAL);
#endif

	case DDIOCSDBG:
		return(dbg_ioctl((void *) arg, DBG_INET));

	case SIOCADDRT: case SIOCADDRTOLD:
	case SIOCDELRT: case SIOCDELRTOLD:
		return(rt_ioctl(cmd,(void *) arg));

	case SIOCDARP:
	case SIOCGARP:
	case SIOCSARP:
		return(arp_ioctl(cmd,(void *) arg));

	case IP_SET_DEV:
	case SIOCGIFCONF:
	case SIOCGIFFLAGS:
	case SIOCSIFFLAGS:
	case SIOCGIFADDR:
	case SIOCSIFADDR:
	case SIOCGIFDSTADDR:
	case SIOCSIFDSTADDR:
	case SIOCGIFBRDADDR:
	case SIOCSIFBRDADDR:
	case SIOCGIFNETMASK:
	case SIOCSIFNETMASK:
	case SIOCGIFMETRIC:
	case SIOCSIFMETRIC:
	case SIOCGIFMEM:
	case SIOCSIFMEM:
	case SIOCGIFMTU:
	case SIOCSIFMTU:
	case SIOCSIFLINK:
	case SIOCGIFHWADDR:
		return(dev_ioctl(cmd,(void *) arg));

	default:
		if (!sk || !sk->prot->ioctl) return(-EINVAL);
		return(sk->prot->ioctl(sk, cmd, arg));
  }
  /*NOTREACHED*/
  return(0);
}

int rt_ioctl(unsigned int cmd, void *arg)
{
	int err;
	struct rtentry rt;

	switch(cmd) {
	case DDIOCSDBG:
		return dbg_ioctl(arg, DBG_RT);
	case SIOCADDRTOLD:
	case SIOCDELRTOLD:
		if (!suser())
			return -EPERM;
		err = get_old_rtent((struct old_rtentry *) arg, &rt);
		if (err)
			return err;
		return (cmd == SIOCDELRTOLD) ? rt_kill(&rt) : rt_new(&rt);
	case SIOCADDRT:
	case SIOCDELRT:
		if (!suser())
			return -EPERM;
		err=verify_area(VERIFY_READ, arg, sizeof(struct rtentry));
		if (err)
			return err;
		memcpy_fromfs(&rt, arg, sizeof(struct rtentry));
		return (cmd == SIOCDELRT) ? rt_kill(&rt) : rt_new(&rt);
	}

	return -EINVAL;
}
```
## Route

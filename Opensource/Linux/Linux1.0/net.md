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
### Route command

- route add -net 192.168.4.0 netmask 255.255.255.0 gw 192.168.1.1
- route add -net 192.168.4.0 netmask 255.255.255.0 pon_d4097
- route add default gw 192.168.1.10
- route del default gw 192.168.1.10

```c
static int rt_new(struct rtentry *r)
{

	if ((devname = r->rt_dev) != NULL) {
		err = getname(devname, &devname);
		if (err)
			return err;
		dev = dev_get(devname);
		putname(devname);
		if (!dev)
			return -EINVAL;
	}
	...
	flags = r->rt_flags;
	daddr = ((struct sockaddr_in *) &r->rt_dst)->sin_addr.s_addr;
	mask = ((struct sockaddr_in *) &r->rt_genmask)->sin_addr.s_addr;
	gw = ((struct sockaddr_in *) &r->rt_gateway)->sin_addr.s_addr;
	...
	rt_add(flags, daddr, mask, gw, dev);
	return 0;
}

void rt_add(short flags, unsigned long dst, unsigned long mask,
	unsigned long gw, struct device *dev)
{
	...
	/* Allocate an entry. */
	rt = (struct rtable *) kmalloc(sizeof(struct rtable), GFP_ATOMIC);
	if (rt == NULL) {
		DPRINTF((DBG_RT, "RT: no memory for new route!\n"));
		return;
	}
	memset(rt, 0, sizeof(struct rtable));
	rt->rt_flags = flags | RTF_UP;
	rt->rt_dst = dst;
	rt->rt_dev = dev;
	rt->rt_gateway = gw;
	rt->rt_mask = mask;
	rt->rt_mtu = dev->mtu;
	rt_print(rt);
	/*
	 * What we have to do is loop though this until we have
	 * found the first address which has a higher generality than
	 * the one in rt.  Then we can put rt in right before it.
	 */
	save_flags(cpuflags);
	cli();
	/* remove old route if we are getting a duplicate. */
	rp = &rt_base;
	while ((r = *rp) != NULL) {
		if (r->rt_dst != dst) {
			rp = &r->rt_next;
			continue;
		}
		*rp = r->rt_next;
		if (rt_loopback == r)
			rt_loopback = NULL;
		kfree_s(r, sizeof(struct rtable));
	}
	/* add the new route */
	rp = &rt_base;
	while ((r = *rp) != NULL) {
		if ((r->rt_mask & mask) != mask)
			break;
		rp = &r->rt_next;
	}
	rt->rt_next = r;
	*rp = rt;
	if (rt->rt_dev->flags & IFF_LOOPBACK)
		rt_loopback = rt;
	restore_flags(cpuflags);
	return;
}

// Delete one route rule

static int rt_kill(struct rtentry *r)
{
	struct sockaddr_in *trg;

	trg = (struct sockaddr_in *) &r->rt_dst;
	rt_del(trg->sin_addr.s_addr);
	return 0;
}

static void rt_del(unsigned long dst)
{
	struct rtable *r, **rp;
	unsigned long flags;

	DPRINTF((DBG_RT, "RT: flushing for dst %s\n", in_ntoa(dst)));
	rp = &rt_base;
	save_flags(flags);
	cli();
	while((r = *rp) != NULL) {
		if (r->rt_dst != dst) {
			rp = &r->rt_next;
			continue;
		}
		*rp = r->rt_next;
		if (rt_loopback == r)
			rt_loopback = NULL;
		kfree_s(r, sizeof(struct rtable));
	} 
	restore_flags(flags);
}

```

Below part aims to make use of Route table to help to find the destination.

In two scenairos, we need the Route table to find us find the destination.
- 多网卡情况下，一个网卡的报文需要通过三层转发到另外一个网卡。
- 多网卡情况下，本机要发送的报文在三层需要通过路由模块找到合适的出口。

```c
static void
ip_forward(struct sk_buff *skb, struct device *dev, int is_frag)
{
	rt = rt_route(iph->daddr, NULL);
}

int
ip_build_header(struct sk_buff *skb, unsigned long saddr, unsigned long daddr,
		struct device **dev, int type, struct options *opt, int len, int tos, int ttl)
{
	...
	if (*dev == NULL) {
	rt = rt_route(daddr, &optmem);
	if (rt == NULL) 
		return(-ENETUNREACH);

	*dev = rt->rt_dev;
	if (saddr == 0x0100007FL && daddr != 0x0100007FL) 
		saddr = rt->rt_dev->pa_addr;
	raddr = rt->rt_gateway;

	DPRINTF((DBG_IP, "ip_build_header: saddr set to %s\n", in_ntoa(saddr)));
	opt = &optmem;
  } else {
	/* We still need the address of the first hop. */
	rt = rt_route(daddr, &optmem);
	raddr = (rt == NULL) ? 0 : rt->rt_gateway;
  }
  ...
}
```
```c
struct rtable * rt_route(unsigned long daddr, struct options *opt)
{
	struct rtable *rt;

	for (rt = rt_base; rt != NULL || early_out ; rt = rt->rt_next) {
		if (!((rt->rt_dst ^ daddr) & rt->rt_mask))
			break;
		/* broadcast addresses can be special cases.. */
		if ((rt->rt_dev->flags & IFF_BROADCAST) &&
		     rt->rt_dev->pa_brdaddr == daddr)
			break;
	}
	if (daddr == rt->rt_dev->pa_addr) {
		if ((rt = rt_loopback) == NULL)
			goto no_route;
	}
	rt->rt_use++;
	return rt;
no_route:
	return NULL;
}
```

## ARP
```c

static int
udp_send(struct sock *sk, struct sockaddr_in *sin,
	 unsigned char *from, int len)
{
  tmp = sk->prot->build_header(skb, saddr, sin->sin_addr.s_addr,
			       &dev, IPPROTO_UDP, sk->opt, skb->mem_len,sk->ip_tos,sk->ip_ttl);
  sk->prot->queue_xmit(sk, dev, skb, 1);
  
  return (len);
}


static int
ip_send(struct sk_buff *skb, unsigned long daddr, int len, struct device *dev,
	unsigned long saddr)
{
  unsigned char *ptr;
  int mac;

  ptr = skb->data;
  mac = 0;
  skb->arp = 1;
  if (dev->hard_header) {
	mac = dev->hard_header(ptr, dev, ETH_P_IP, daddr, saddr, len);
  }
  if (mac < 0) {
	mac = -mac;
	skb->arp = 0;
  }
  skb->dev = dev;
  return(mac);
}
int
ip_build_header(struct sk_buff *skb, unsigned long saddr, unsigned long daddr,
		struct device **dev, int type, struct options *opt, int len, int tos, int ttl)
{
	// First Step: we need to get the route table to find the interface which we will make use of to send the frame.
	rt = rt_route(daddr, &optmem);
	raddr = (rt == NULL) ? 0 : rt->rt_gateway;
	
	// Second Step: we need to build MAC header, dstintaion MAC  
    tmp = ip_send(skb, raddr, len, *dev, saddr);
	
}

static int
ip_send(struct sk_buff *skb, unsigned long daddr, int len, struct device *dev,
	unsigned long saddr)
{
  unsigned char *ptr;
  int mac;

  ptr = skb->data;
  mac = 0;
  skb->arp = 1;
  if (dev->hard_header) {
	mac = dev->hard_header(ptr, dev, ETH_P_IP, daddr, saddr, len);
  }
  //If mac < 0. it means we find no arp entry, will send arp request to get it.
  if (mac < 0) {
	mac = -mac;
	skb->arp = 0;
  }
  skb->dev = dev;
  return(mac);
}

int
eth_header(unsigned char *buff, struct device *dev, unsigned short type,
	   unsigned long daddr, unsigned long saddr, unsigned len)
{
	
  memcpy(eth->h_source, &saddr, 4);
  /* No. Ask ARP to resolve the Ethernet address. */
  if (arp_find(eth->h_dest, daddr, dev, dev->pa_addr)) 
  {
        sti();
        if(type!=ETH_P_IP)
        	printk("Erk: protocol %X got into an arp request state!\n",type);
	return(-dev->hard_header_len);
  } 
  else
  {
    //Find the arp entry
  	memcpy(eth->h_source,dev->dev_addr,dev->addr_len);	/* This was missing causing chaos if the
  								   header built correctly! */
  	sti();
  	return(dev->hard_header_len);
  }
}



```

**Note : In arp related thing, we will met some packets wait for dst mac**
```c
int
arp_rcv(struct sk_buff *skb, struct device *dev, struct packet_type *pt)
{
  //Check this arp accoring to the existing arp table.
  tbl = arp_lookup(src);
  if (tbl != NULL) {
	DPRINTF((DBG_ARP, "ARP: udating entry for %s\n", in_ntoa(src)));
	memcpy(tbl->ha, ptr, arp->ar_hln);
	tbl->hlen = arp->ar_hln;
	tbl->flags |= ATF_COM;
	tbl->last_used = jiffies;
  } else {
	memcpy(&dst, ptr + (arp->ar_hln * 2) + arp->ar_pln, arp->ar_pln);
	if (chk_addr(dst) != IS_MYADDR && arp_proxies == 0) {
		kfree_skb(skb, FREE_READ);
		return(0);
	} else {
		tbl = arp_create(src, ptr, arp->ar_hln, arp->ar_hrd);
		if (tbl == NULL) {
			kfree_skb(skb, FREE_READ);
			return(0);
		}
	}
  }
  //After arp table updates, we need to check the blocking packets which wait for the dstination MAC.
	arp_send_q();
}

static void
arp_send_q(void)
{
  struct sk_buff *skb;
  struct sk_buff *volatile work_q;
  cli();
  work_q = arp_q;
  skb_new_list_head(&work_q);
  arp_q = NULL;
  sti();
  while((skb=skb_dequeue(&work_q))!=NULL)
  {
  	IS_SKB(skb);
	skb->magic = 0;
	skb->next = NULL;
	skb->prev = NULL;

	/* Decrement the 'tries' counter. */
	cli();
	skb->tries--;
	if (skb->tries == 0) {
		/*
		 * Grmpf.
		 * We have tried ARP_MAX_TRIES to resolve the IP address
		 * from this datagram.  This means that the machine does
		 * not listen to our ARP requests.  Perhaps someone tur-
		 * ned off the thing?
		 * In any case, trying further is useless.  So, we kill
		 * this packet from the queue.  (grinnik) -FvK
		 */
		skb->sk = NULL;
		if(skb->free)
			kfree_skb(skb, FREE_WRITE);
			/* If free was 0, magic is now 0, next is 0 and 
			   the write queue will notice and kill */
		sti();
		continue;
	}

	/* Can we now complete this packet? */
	sti();
	if (skb->arp || !skb->dev->rebuild_header(skb->data, skb->dev)) {
		skb->arp  = 1;
		skb->dev->queue_xmit(skb, skb->dev, 0);
	} else {
		/* Alas.  Re-queue it... */
		skb->magic = ARP_QUEUE_MAGIC;      
		skb_queue_head(&arp_q,skb);
	}
  }
}


dev_queue_xmit(struct sk_buff *skb, struct device *dev, int pri)
{
  // If the device is busy, return 1.
  if (dev->hard_start_xmit(skb, dev) == 0) {
	return;
  }

  /* Put skb into a bidirectional circular linked list. */
  DPRINTF((DBG_DEV, "dev_queue_xmit dev->buffs[%d]=%X\n",
					pri, dev->buffs[pri]));

  /* Interrupts should already be cleared by hard_start_xmit. */
  cli();
  skb->magic = DEV_QUEUE_MAGIC;
  if(where)
  	skb_queue_head(&dev->buffs[pri],skb);
  else
  	skb_queue_tail(&dev->buffs[pri],skb);
  skb->magic = DEV_QUEUE_MAGIC;
}

```
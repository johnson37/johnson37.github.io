# vconfig

vconfig is one busybox tool to configure vlan on net device.

## code flow
### Application Layer
```c
int vconfig_main(int argc, char **argv)
{
    fd = xsocket(AF_INET, SOCK_STREAM, 0);
    ioctl_or_perror_and_die(fd, SIOCSIFVLAN, &ifr,
                        "ioctl error for %s", *argv);

}


int FAST_FUNC ioctl_or_perror_and_die(int fd, unsigned request, void *argp, const char *fmt,...)
{
    int ret;
    va_list p;

    ret = ioctl(fd, request, argp);
    return ret;
}

```
### Kernel Layer
```c
static int compat_sock_ioctl_trans(struct file *file, struct socket *sock,
             unsigned int cmd, unsigned long arg)
{
    case SIOCSIFVLAN:
        return sock_ioctl(file, cmd, arg);
}
static long sock_ioctl(struct file *file, unsigned cmd, unsigned long arg)
{
    switch (cmd) {
        case SIOCGIFVLAN:
        case SIOCSIFVLAN:
            err = -ENOPKG;
            if (!vlan_ioctl_hook)
                request_module("8021q");

            mutex_lock(&vlan_ioctl_mutex);
            if (vlan_ioctl_hook)
                err = vlan_ioctl_hook(net, argp);
            mutex_unlock(&vlan_ioctl_mutex);
            break;
	
	}
}

static int vlan_ioctl_handler(struct net *net, void __user *arg)
{
    switch (args.cmd) {
    case ADD_VLAN_CMD:
        err = -EPERM;
        if (!ns_capable(net->user_ns, CAP_NET_ADMIN))
            break;
        err = register_vlan_device(dev, args.u.VID);
        break;
	
	}
}

static int register_vlan_device(struct net_device *real_dev, u16 vlan_id)
{
    switch (vn->name_type) {
    case VLAN_NAME_TYPE_RAW_PLUS_VID_NO_PAD:
        /* Put our vlan.VID in the name.
         * Name will look like:  eth0.5
         */
        snprintf(name, IFNAMSIZ, "%s.%i", real_dev->name, vlan_id);
        break;
		
	}
    new_dev = alloc_netdev(sizeof(struct vlan_dev_priv), name,
                   NET_NAME_UNKNOWN, vlan_setup);
	
}

```

### Packet flow
```c
static int __netif_receive_skb(struct sk_buff *skb)
{
    ret = __netif_receive_skb_core(skb, false);
}

static int __netif_receive_skb_core(struct sk_buff *skb, bool pfmemalloc)
{
    if (skb_vlan_tag_present(skb)) {
        if (pt_prev) {
            ret = deliver_skb(skb, pt_prev, orig_dev);
            pt_prev = NULL;
        }
        if (vlan_do_receive(&skb))
            goto another_round;
        else if (unlikely(!skb))
            goto out;
    }
}
bool vlan_do_receive(struct sk_buff **skbp)
{
    vlan_dev = vlan_find_dev(skb->dev, vlan_proto, vlan_id);
    skb->dev = vlan_dev;
    skb->priority = vlan_get_ingress_priority(vlan_dev, skb->vlan_tci);
    skb->vlan_tci = 0;

}
```

## chart
[chart](./pic/vconfig.PNG)
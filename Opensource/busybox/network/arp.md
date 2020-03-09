# arp

arp command 默认情况下是display当前存在的arp表项，也可以通过arp -s/-d 去增加或者删除一条表项。

## display
display的实现直接从/proc/net/arp中得到。

## set 增加或者删除表项
通过ioctl 调用实现。

## code Flow

### Application Layer

```c
int arp_main(int argc UNUSED_PARAM, char **argv)
{
    /* Now see what we have to do here... */
    if (opts & (ARP_OPT_d | ARP_OPT_s)) {
        if (argv[0] == NULL)
            bb_error_msg_and_die("need host name");
        if (opts & ARP_OPT_s)
            return arp_set(argv);
        return arp_del(argv);
    }
    //if (opts & ARP_OPT_a) - default
    return arp_show(argv[0]);

}
```

### Kernel
```c
int arp_ioctl(struct net *net, unsigned int cmd, void __user *arg)
{
    switch (cmd) {
    case SIOCDARP:
        err = arp_req_delete(net, &r, dev);
        break;
    case SIOCSARP:
        err = arp_req_set(net, &r, dev);
        break;
    case SIOCGARP:
        err = arp_req_get(&r, dev);
        break;
    }

}

static int arp_req_set(struct net *net, struct arpreq *r,
               struct net_device *dev)
{
    neigh = __neigh_lookup_errno(&arp_tbl, &ip, dev);
    err = PTR_ERR(neigh);
    if (!IS_ERR(neigh)) {
        unsigned int state = NUD_STALE;
        if (r->arp_flags & ATF_PERM)
            state = NUD_PERMANENT;
        err = neigh_update(neigh, (r->arp_flags & ATF_COM) ?
                   r->arp_ha.sa_data : NULL, state,
                   NEIGH_UPDATE_F_OVERRIDE |
                   NEIGH_UPDATE_F_ADMIN);
        neigh_release(neigh);
    }

}

static inline struct neighbour *
__neigh_lookup_errno(struct neigh_table *tbl, const void *pkey,
  struct net_device *dev)
{
    struct neighbour *n = neigh_lookup(tbl, pkey, dev);

    if (n)
        return n;

    return neigh_create(tbl, pkey, dev);
}


static int arp_req_delete(struct net *net, struct arpreq *r,
              struct net_device *dev)
{
    return arp_invalidate(dev, ip);
}

static int arp_invalidate(struct net_device *dev, __be32 ip)
{
    struct neighbour *neigh = neigh_lookup(&arp_tbl, &ip, dev);
    int err = -ENXIO;

    if (neigh) {
        if (neigh->nud_state & ~NUD_NOARP)
            err = neigh_update(neigh, NULL, NUD_FAILED,
                       NEIGH_UPDATE_F_OVERRIDE|
                       NEIGH_UPDATE_F_ADMIN);
        neigh_release(neigh);
    }

    return err;
}

```
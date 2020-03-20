# route

## Application Layer

```c
int route_main(int argc UNUSED_PARAM, char **argv)
{
    INET_setroute(what, argv);
}

static NOINLINE void INET_setroute(int action, char **args)
{
    /* Create a socket to the INET kernel. */
    skfd = xsocket(AF_INET, SOCK_DGRAM, 0);

    if (action == RTACTION_ADD)
        xioctl(skfd, SIOCADDRT, &rt);
    else
        xioctl(skfd, SIOCDELRT, &rt);

    if (ENABLE_FEATURE_CLEAN_UP) close(skfd);
}
```

## Kernel Layer
```c
int inet_ioctl(struct socket *sock, unsigned int cmd, unsigned long arg)
{
    struct sock *sk = sock->sk;
    int err = 0;
    struct net *net = sock_net(sk);

    switch (cmd) {
    case SIOCGSTAMP:
        err = sock_get_timestamp(sk, (struct timeval __user *)arg);
        break;
    case SIOCGSTAMPNS:
        err = sock_get_timestampns(sk, (struct timespec __user *)arg);
        break;
    case SIOCADDRT:
    case SIOCDELRT:
    case SIOCRTMSG:
        err = ip_rt_ioctl(net, cmd, (void __user *)arg);
        break;
	}
}

int ip_rt_ioctl(struct net *net, unsigned int cmd, void __user *arg)
{
    struct fib_config cfg;
    struct rtentry rt;
    int err;

    switch (cmd) {
    case SIOCADDRT:     /* Add a route */
    case SIOCDELRT:     /* Delete a route */
        if (!ns_capable(net->user_ns, CAP_NET_ADMIN))
            return -EPERM;

        if (copy_from_user(&rt, arg, sizeof(rt)))
            return -EFAULT;
        
        rtnl_lock();
        err = rtentry_to_fib_config(net, cmd, &rt, &cfg);
        if (err == 0) {
            struct fib_table *tb;
            
            if (cmd == SIOCDELRT) {
                tb = fib_get_table(net, cfg.fc_table);
                if (tb) 
                    err = fib_table_delete(tb, &cfg);
                else
                    err = -ESRCH;
            } else { 
                tb = fib_new_table(net, cfg.fc_table);
                if (tb) 
                    err = fib_table_insert(tb, &cfg);
                else
                    err = -ENOBUFS;
            }

            /* allocated by rtentry_to_fib_config() */
            kfree(cfg.fc_mx);
        }
        rtnl_unlock();
        return err;
    }
    return -EINVAL;
}

struct fib_table *fib_new_table(struct net *net, u32 id)
{       
    struct fib_table *tb, *alias = NULL;
    unsigned int h;
        
    if (id == 0)
        id = RT_TABLE_MAIN;
    tb = fib_get_table(net, id);
    if (tb)
        return tb; 
                
    if (id == RT_TABLE_LOCAL && !net->ipv4.fib_has_custom_rules)
        alias = fib_new_table(net, RT_TABLE_MAIN);
                    
    tb = fib_trie_table(id, alias);
    if (!tb)    
        return NULL;
                    
    switch (id) {
    case RT_TABLE_LOCAL:
        rcu_assign_pointer(net->ipv4.fib_local, tb);
        break;
    case RT_TABLE_MAIN:
        rcu_assign_pointer(net->ipv4.fib_main, tb);
        break;
    case RT_TABLE_DEFAULT:
        rcu_assign_pointer(net->ipv4.fib_default, tb);
        break;
    default:
        break;
    }

    h = id & (FIB_TABLE_HASHSZ - 1);
    hlist_add_head_rcu(&tb->tb_hlist, &net->ipv4.fib_table_hash[h]);
    return tb;
}


```
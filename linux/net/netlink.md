# netlink

## Basic Introduction

## code flow


### normal netlink

#### Register
```c
core_initcall(netlink_proto_init);

static int __init netlink_proto_init(void)
{
    rtnetlink_init();
}

void __init rtnetlink_init(void)
{
    if (register_pernet_subsys(&rtnetlink_net_ops))
        panic("rtnetlink_init: cannot initialize rtnetlink\n");

    register_netdevice_notifier(&rtnetlink_dev_notifier);

    rtnl_register(PF_UNSPEC, RTM_GETLINK, rtnl_getlink,
              rtnl_dump_ifinfo, rtnl_calcit);
    rtnl_register(PF_UNSPEC, RTM_SETLINK, rtnl_setlink, NULL, NULL);
    rtnl_register(PF_UNSPEC, RTM_NEWLINK, rtnl_newlink, NULL, NULL);
    rtnl_register(PF_UNSPEC, RTM_DELLINK, rtnl_dellink, NULL, NULL);

    rtnl_register(PF_UNSPEC, RTM_GETADDR, NULL, rtnl_dump_all, NULL);
    rtnl_register(PF_UNSPEC, RTM_GETROUTE, NULL, rtnl_dump_all, NULL);

    rtnl_register(PF_BRIDGE, RTM_NEWNEIGH, rtnl_fdb_add, NULL, NULL);
    rtnl_register(PF_BRIDGE, RTM_DELNEIGH, rtnl_fdb_del, NULL, NULL);
    rtnl_register(PF_BRIDGE, RTM_GETNEIGH, NULL, rtnl_fdb_dump, NULL);

    rtnl_register(PF_BRIDGE, RTM_GETLINK, NULL, rtnl_bridge_getlink, NULL);
    rtnl_register(PF_BRIDGE, RTM_DELLINK, rtnl_bridge_dellink, NULL, NULL);
    rtnl_register(PF_BRIDGE, RTM_SETLINK, rtnl_bridge_setlink, NULL, NULL);
}


static struct pernet_operations rtnetlink_net_ops = {
    .init = rtnetlink_net_init,
    .exit = rtnetlink_net_exit,
};

static int __net_init rtnetlink_net_init(struct net *net)
{
    struct sock *sk;
    struct netlink_kernel_cfg cfg = {
        .groups     = RTNLGRP_MAX,
        .input      = rtnetlink_rcv,
        .cb_mutex   = &rtnl_mutex,
        .flags      = NL_CFG_F_NONROOT_RECV,
    };
    
    sk = netlink_kernel_create(net, NETLINK_ROUTE, &cfg);
    if (!sk)
        return -ENOMEM;
    net->rtnl = sk;
    return 0;
}

static inline struct sock *
netlink_kernel_create(struct net *net, int unit, struct netlink_kernel_cfg *cfg)
{
    return __netlink_kernel_create(net, unit, THIS_MODULE, cfg);
}


struct sock *
__netlink_kernel_create(struct net *net, int unit, struct module *module,
            struct netlink_kernel_cfg *cfg)
{
    if (cfg && cfg->input)
        nlk_sk(sk)->netlink_rcv = cfg->input;

}
```

#### packet process
```c
static int netlink_sendmsg(struct socket *sock, struct msghdr *msg, size_t len)
{
    err = netlink_unicast(sk, skb, dst_portid, msg->msg_flags&MSG_DONTWAIT);
}

int netlink_unicast(struct sock *ssk, struct sk_buff *skb,
		    u32 portid, int nonblock)
{
	struct sock *sk;
	int err;
	long timeo;

	skb = netlink_trim(skb, gfp_any());

	timeo = sock_sndtimeo(ssk, nonblock);
retry:
	sk = netlink_getsockbyportid(ssk, portid);
	if (IS_ERR(sk)) {
		kfree_skb(skb);
		return PTR_ERR(sk);
	}
	if (netlink_is_kernel(sk))
		return netlink_unicast_kernel(sk, skb, ssk);

	if (sk_filter(sk, skb)) {
		err = skb->len;
		kfree_skb(skb);
		sock_put(sk);
		return err;
	}

	err = netlink_attachskb(sk, skb, &timeo, ssk);
	if (err == 1)
		goto retry;
	if (err)
		return err;

	return netlink_sendskb(sk, skb);
}

static int netlink_unicast_kernel(struct sock *sk, struct sk_buff *skb,
				  struct sock *ssk)
{
	int ret;
	struct netlink_sock *nlk = nlk_sk(sk);

	ret = -ECONNREFUSED;
	if (nlk->netlink_rcv != NULL) {
		ret = skb->len;
		netlink_skb_set_owner_r(skb, sk);
		NETLINK_CB(skb).sk = ssk;
		netlink_deliver_tap_kernel(sk, ssk, skb);
		nlk->netlink_rcv(skb);
		consume_skb(skb);
	} else {
		kfree_skb(skb);
	}
	sock_put(sk);
	return ret;
}


static void rtnetlink_rcv(struct sk_buff *skb)
{
	rtnl_lock();
	netlink_rcv_skb(skb, &rtnetlink_rcv_msg);
	rtnl_unlock();
}

static int rtnetlink_rcv_msg(struct sk_buff *skb, struct nlmsghdr *nlh)
{
	doit = rtnl_get_doit(family, type);
	if (doit == NULL)
		return -EOPNOTSUPP;

	return doit(skb, nlh);
}
```

### family netlink 

#### Register

```c
subsys_initcall(genl_init);

static int __init genl_init(void)
{   
    int i, err;

    for (i = 0; i < GENL_FAM_TAB_SIZE; i++)
        INIT_LIST_HEAD(&family_ht[i]);

    err = genl_register_family_with_ops_groups(&genl_ctrl, genl_ctrl_ops,
                           genl_ctrl_groups);
    if (err < 0)
        goto problem;

    err = register_pernet_subsys(&genl_pernet_ops);
    if (err)
        goto problem;

    return 0;

problem:
    panic("GENL: Cannot register controller: %d\n", err);
}

static struct pernet_operations genl_pernet_ops = {
    .init = genl_pernet_init,
    .exit = genl_pernet_exit,
}; 

static int __net_init genl_pernet_init(struct net *net)
{           
    struct netlink_kernel_cfg cfg = {
        .input      = genl_rcv,
        .flags      = NL_CFG_F_NONROOT_RECV,
        .bind       = genl_bind,
        .unbind     = genl_unbind,
    };  
        
    /* we'll bump the group number right afterwards */
    net->genl_sock = netlink_kernel_create(net, NETLINK_GENERIC, &cfg);

    if (!net->genl_sock && net_eq(net, &init_net))
        panic("GENL: Cannot initialize generic netlink\n");
              
    if (!net->genl_sock)
        return -ENOMEM;

    return 0;   
}               


struct sock *
__netlink_kernel_create(struct net *net, int unit, struct module *module,
            struct netlink_kernel_cfg *cfg)
{
    if (cfg && cfg->input)
        nlk_sk(sk)->netlink_rcv = cfg->input;

}
```


#### Packet process
```c
static int netlink_sendmsg(struct socket *sock, struct msghdr *msg, size_t len)
{
    err = netlink_unicast(sk, skb, dst_portid, msg->msg_flags&MSG_DONTWAIT);
}

int netlink_unicast(struct sock *ssk, struct sk_buff *skb,
		    u32 portid, int nonblock)
{
	struct sock *sk;
	int err;
	long timeo;

	skb = netlink_trim(skb, gfp_any());

	timeo = sock_sndtimeo(ssk, nonblock);
retry:
	sk = netlink_getsockbyportid(ssk, portid);
	if (IS_ERR(sk)) {
		kfree_skb(skb);
		return PTR_ERR(sk);
	}
	if (netlink_is_kernel(sk))
		return netlink_unicast_kernel(sk, skb, ssk);

	if (sk_filter(sk, skb)) {
		err = skb->len;
		kfree_skb(skb);
		sock_put(sk);
		return err;
	}

	err = netlink_attachskb(sk, skb, &timeo, ssk);
	if (err == 1)
		goto retry;
	if (err)
		return err;

	return netlink_sendskb(sk, skb);
}

static int netlink_unicast_kernel(struct sock *sk, struct sk_buff *skb,
				  struct sock *ssk)
{
	int ret;
	struct netlink_sock *nlk = nlk_sk(sk);

	ret = -ECONNREFUSED;
	if (nlk->netlink_rcv != NULL) {
		ret = skb->len;
		netlink_skb_set_owner_r(skb, sk);
		NETLINK_CB(skb).sk = ssk;
		netlink_deliver_tap_kernel(sk, ssk, skb);
		nlk->netlink_rcv(skb);
		consume_skb(skb);
	} else {
		kfree_skb(skb);
	}
	sock_put(sk);
	return ret;
}

static void genl_rcv(struct sk_buff *skb)
{
	down_read(&cb_lock);
	netlink_rcv_skb(skb, &genl_rcv_msg);
	up_read(&cb_lock);
}

int netlink_rcv_skb(struct sk_buff *skb, int (*cb)(struct sk_buff *,
                             struct nlmsghdr *))
{
    struct nlmsghdr *nlh;
    int err;

    while (skb->len >= nlmsg_total_size(0)) {
        int msglen;          

        nlh = nlmsg_hdr(skb);
        err = 0;

        if (nlh->nlmsg_len < NLMSG_HDRLEN || skb->len < nlh->nlmsg_len)
            return 0;
    
        /* Only requests are handled by the kernel */
        if (!(nlh->nlmsg_flags & NLM_F_REQUEST))
            goto ack;
        
        /* Skip control messages */
        if (nlh->nlmsg_type < NLMSG_MIN_TYPE)
            goto ack;
        
        err = cb(skb, nlh);
        if (err == -EINTR)
            goto skip;
        
ack:        
        if (nlh->nlmsg_flags & NLM_F_ACK || err)
            netlink_ack(skb, nlh, err);

skip:
        msglen = NLMSG_ALIGN(nlh->nlmsg_len);
        if (msglen > skb->len)
            msglen = skb->len;
        skb_pull(skb, msglen);
    }

    return 0;
}

static int genl_rcv_msg(struct sk_buff *skb, struct nlmsghdr *nlh)
{
	struct genl_family *family;
	int err;

	family = genl_family_find_byid(nlh->nlmsg_type);
	if (family == NULL)
		return -ENOENT;

	if (!family->parallel_ops)
		genl_lock();

	err = genl_family_rcv_msg(family, skb, nlh);

	if (!family->parallel_ops)
		genl_unlock();

	return err;
}
```
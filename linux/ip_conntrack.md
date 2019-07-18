# IP Conntrack

## Basic Introduction
ip conntrack 是iptable中的一部分，用于监控网络中行程的链接，比如TCP的链接以及UDP的链接以及每条链接的状态。

## 
```c
[root@sfu01 net]# pwd
/proc/net
[root@sfu01 net]# cat nf_conntrack
ipv4     2 udp      17 2 src=127.0.0.1 dst=127.0.0.1 sport=51873 dport=53 [UNREPLIED] src=127.0.0.1 dst=127.0.0.1 sport=53 dport=51873 mark=0 secmark=0 use=2
ipv4     2 udp      17 16 src=135.242.61.131 dst=135.242.61.255 sport=137 dport=137 [UNREPLIED] src=135.242.61.255 dst=135.242.61.131 sport=137 dport=137 mark=0 secmark=0 use=2
ipv4     2 tcp      6 299 ESTABLISHED src=135.242.61.36 dst=135.242.61.59 sport=58292 dport=22 src=135.242.61.59 dst=135.242.61.36 sport=22 dport=58292 [ASSURED] mark=0 secmark=0 use=2
ipv4     2 udp      17 10 src=135.242.61.148 dst=135.242.61.255 sport=52173 dport=1947 [UNREPLIED] src=135.242.61.255 dst=135.242.61.148 sport=1947 dport=52173 mark=0 secmark=0 use=2
ipv4     2 udp      17 18 src=135.242.61.210 dst=135.242.61.255 sport=138 dport=138 [UNREPLIED] src=135.242.61.255 dst=135.242.61.210 sport=138 dport=138 mark=0 secmark=0 use=2
ipv4     2 udp      17 2 src=135.242.61.126 dst=135.242.61.255 sport=57326 dport=1947 [UNREPLIED] src=135.242.61.255 dst=135.242.61.126 sport=1947 dport=57326 mark=0 secmark=0 use=2
ipv4     2 udp      17 0 src=135.242.61.147 dst=135.242.61.255 sport=137 dport=137 [UNREPLIED] src=135.242.61.255 dst=135.242.61.147 sport=137 dport=137 mark=0 secmark=0 use=2
ipv4     2 udp      17 2 src=127.0.0.1 dst=127.0.0.1 sport=52855 dport=53 [UNREPLIED] src=127.0.0.1 dst=127.0.0.1 sport=53 dport=52855 mark=0 secmark=0 use=2
ipv4     2 tcp      6 431960 ESTABLISHED src=10.242.147.54 dst=135.242.61.59 sport=64634 dport=22 src=135.242.61.59 dst=10.242.147.54 sport=22 dport=64634 [ASSURED] mark=0 secmark=0 use=2
ipv4     2 udp      17 2 src=135.242.61.148 dst=255.255.255.255 sport=52173 dport=1947 [UNREPLIED] src=255.255.255.255 dst=135.242.61.148 sport=1947 dport=52173 mark=0 secmark=0 use=2
ipv4     2 udp      17 7 src=135.242.61.150 dst=135.242.61.255 sport=138 dport=138 [UNREPLIED] src=135.242.61.255 dst=135.242.61.150 sport=138 dport=138 mark=0 secmark=0 use=2
ipv4     2 udp      17 19 src=0.0.0.0 dst=255.255.255.255 sport=68 dport=67 [UNREPLIED] src=255.255.255.255 dst=0.0.0.0 sport=67 dport=68 mark=0 secmark=0 use=2
ipv4     2 tcp      6 75 TIME_WAIT src=135.242.61.36 dst=135.242.61.59 sport=58290 dport=22 src=135.242.61.59 dst=135.242.61.36 sport=22 dport=58290 [ASSURED] mark=0 secmark=0 use=2
ipv4     2 udp      17 14 src=135.242.61.143 dst=135.242.61.255 sport=137 dport=137 [UNREPLIED] src=135.242.61.255 dst=135.242.61.143 sport=137 dport=137 mark=0 secmark=0 use=2
ipv4     2 udp      17 23 src=135.242.61.158 dst=135.242.61.255 sport=137 dport=137 [UNREPLIED] src=135.242.61.255 dst=135.242.61.158 sport=137 dport=137 mark=0 secmark=0 use=2
ipv4     2 udp      17 3 src=135.242.61.157 dst=255.255.255.255 sport=1094 dport=1092 [UNREPLIED] src=255.255.255.255 dst=135.242.61.157 sport=1092 dport=1094 mark=0 secmark=0 use=2

```

## Code Flow
ip_conntrack 也是基于Netfilter架构。

```c
static struct nf_hook_ops ipv4_conntrack_ops[] __read_mostly = {
    {
        .hook       = ipv4_conntrack_in,
        .pf     = NFPROTO_IPV4,
        .hooknum    = NF_INET_PRE_ROUTING,
        .priority   = NF_IP_PRI_CONNTRACK,
    },
    {
        .hook       = ipv4_conntrack_local,
        .pf     = NFPROTO_IPV4,
        .hooknum    = NF_INET_LOCAL_OUT,
        .priority   = NF_IP_PRI_CONNTRACK,
    },
    {
        .hook       = ipv4_helper,
        .pf     = NFPROTO_IPV4,
        .hooknum    = NF_INET_POST_ROUTING,
        .priority   = NF_IP_PRI_CONNTRACK_HELPER,
    },
    {
        .hook       = ipv4_confirm,
        .pf     = NFPROTO_IPV4,
        .hooknum    = NF_INET_POST_ROUTING,
        .priority   = NF_IP_PRI_CONNTRACK_CONFIRM,
    },
    {
        .hook       = ipv4_helper,
        .pf     = NFPROTO_IPV4,
        .hooknum    = NF_INET_LOCAL_IN,
        .priority   = NF_IP_PRI_CONNTRACK_HELPER,
    },
    {
        .hook       = ipv4_confirm,
        .pf     = NFPROTO_IPV4,
        .hooknum    = NF_INET_LOCAL_IN,
        .priority   = NF_IP_PRI_CONNTRACK_CONFIRM,
    },
};

//Packet will pass through NF_INET_PRE_ROUTING/NF_INET_LOCAL_IN/NF_INET_LOCAL_OUT(response message)/NF_INET_POST_ROUTING(response message)
//We will call ipv4_conntrack_in & ipv4_confirm

static unsigned int ipv4_conntrack_in(void *priv,
                      struct sk_buff *skb,
                      const struct nf_hook_state *state)
{
    return nf_conntrack_in(state->net, PF_INET, state->hook, skb);
}

unsigned int
nf_conntrack_in(struct net *net, u_int8_t pf, unsigned int hooknum,
        struct sk_buff *skb)
{

    /* rcu_read_lock()ed by nf_hook_slow */
    l3proto = __nf_ct_l3proto_find(pf);
    ret = l3proto->get_l4proto(skb, skb_network_offset(skb),
                   &dataoff, &protonum);
    l4proto = __nf_ct_l4proto_find(pf, protonum);

    ct = resolve_normal_ct(net, tmpl, skb, dataoff, pf, protonum,
                   l3proto, l4proto, &set_reply, &ctinfo);
	
}

static inline struct nf_conn *
resolve_normal_ct(struct net *net, struct nf_conn *tmpl,
          struct sk_buff *skb,
          unsigned int dataoff,
          u_int16_t l3num,
          u_int8_t protonum,
          struct nf_conntrack_l3proto *l3proto,
          struct nf_conntrack_l4proto *l4proto,
          int *set_reply,
          enum ip_conntrack_info *ctinfo)
{
    h = __nf_conntrack_find_get(net, zone, &tuple, hash);
    if (!h) {
        h = init_conntrack(net, tmpl, &tuple, l3proto, l4proto,
                   skb, dataoff, hash);
        if (!h)
            return NULL;
        if (IS_ERR(h))
            return (void *)h;
    }
    ct = nf_ct_tuplehash_to_ctrack(h);

    skb->nfct = &ct->ct_general;
    skb->nfctinfo = *ctinfo;
 
}

// Now we need to confirm it.
static unsigned int ipv4_confirm(void *priv,
                 struct sk_buff *skb,
                 const struct nf_hook_state *state)
{
	return nf_conntrack_confirm(skb);
}

static inline int nf_conntrack_confirm(struct sk_buff *skb)
{
    struct nf_conn *ct = (struct nf_conn *)skb->nfct;
    int ret = NF_ACCEPT;

    if (ct && !nf_ct_is_untracked(ct)) {
        if (!nf_ct_is_confirmed(ct))
            ret = __nf_conntrack_confirm(skb);
        if (likely(ret == NF_ACCEPT))
            nf_ct_deliver_cached_events(ct);
    }   
    return ret;
}   

int
__nf_conntrack_confirm(struct sk_buff *skb)
{
	__nf_conntrack_hash_insert(ct, hash, reply_hash);
}

```
# IP Conntrack

## Basic Introduction
ip conntrack 是iptable中的一部分，用于监控网络中行程的链接，比如TCP的链接以及UDP的链接以及每条链接的状态。

## 

**Note: 如果看不到，可能是防火墙别关闭，conntrack模块会在开始防火墙时被加载。**
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
## Process procedue
### 格式解析
```c
ipv4     2 udp      17 2 src=127.0.0.1 dst=127.0.0.1 sport=51873 dport=53 [UNREPLIED] src=127.0.0.1 dst=127.0.0.1 sport=53 dport=51873 mark=0 secmark=0 use=2
```
- udp/tcp/icmp 表明协议类型
- 17: 对应协议的协议号
- x: 剩余的存活时间
- src,dst,sport,dport: 连接发出报文的格式
- UNREPLIED: 标志位，UNREPLIED 表示该连接在等待回复。
- src,dst,sport,dport: 连接期望收到的回复报文格式。 

### TCP
对于TCP报文，链接的建立是三次握手的过程。作为连接的发起方，发送SYNC报文之后，连接初始化为SYN_SENT状态，并设置为UNREPLIED标志，当收到对端发送的SYNC+ACK,状态为SYN_RECV，当发送最后的ACK，
状态为ESTABLISH。当该连接有一定的流量时，会设置ASSURED标志。


### UDP
当UDP连接发送第一个报文时，状态为NEW,标志位UNREPLIED，当收到第一个回复报文时，状态位ESTABLISH。当该连接有一定流量时，会有ASSURED标志。

### ICMP
当发送请求时，状态为NEW，收到RESPONSE, 状态切换为ESTABLISH.


## Code Flow
ip_conntrack 也是基于Netfilter架构。

**Note: conntrack有几个主要的函数，ipv4_conntrack_in/ipv4_conntrack_local/ipv4_helper/ipv4_confirm**
**ipv4_conntrack_in/ipv4_conntrack_local --> to build the conntrack**
**ipv4_help:For Linux systems configured with NAT forwarding and firewall functions, 
protocols such as SIP, H323, and ftp cannot work properly. Such protocols generally consist of two types of traffic: 
control flow and data flow, where control flow is used to negotiate data. 
The parameters of the flow include IP address and port information. 
NAT changed the IP address and port number causing its data flow to be blocked. 
The function ipv4_helper is used to handle this situation. 
For example, the helper functions sip_help_udp and sip_help_tcp of the SIP protocol 
are used to modify the media stream negotiation parameters in the SIP protocol.**

**ipv4_confirm: The function ipv4_confirm finally calls __nf_conntrack_confirm for specific processing, 
which removes the previously created connection tracking structure nf_conn from the unconfirmed list and adds it to the global nf_conntrack_hash list. 
See function __nf_conntrack_hash_insert, which adds the two directions of the data stream (IP_CT_DIR_ORIGINAL and IP_CT_DIR_REPLY) 
to the global nf_conntrack_hash list indexed by different hash values ​​(calculated from the stream characteristics).**

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

struct nf_conntrack_l3proto nf_conntrack_l3proto_ipv4 __read_mostly = {
    .l3proto     = PF_INET,
    .name        = "ipv4",
    .pkt_to_tuple    = ipv4_pkt_to_tuple,
    .invert_tuple    = ipv4_invert_tuple,
    .print_tuple     = ipv4_print_tuple,
    .get_l4proto     = ipv4_get_l4proto,
#if IS_ENABLED(CONFIG_NF_CT_NETLINK)
    .tuple_to_nlattr = ipv4_tuple_to_nlattr,
    .nlattr_tuple_size = ipv4_nlattr_tuple_size,
    .nlattr_to_tuple = ipv4_nlattr_to_tuple,
    .nla_policy  = ipv4_nla_policy,
#endif
#if defined(CONFIG_SYSCTL) && defined(CONFIG_NF_CONNTRACK_PROC_COMPAT)
    .ctl_table_path  = "net/ipv4/netfilter",
#endif
    .init_net    = ipv4_init_net,
    .me      = THIS_MODULE,
};

struct nf_conntrack_l3proto nf_conntrack_l3proto_ipv6 __read_mostly = {
    .l3proto        = PF_INET6,
    .name           = "ipv6",
    .pkt_to_tuple       = ipv6_pkt_to_tuple,
    .invert_tuple       = ipv6_invert_tuple,
    .print_tuple        = ipv6_print_tuple,
    .get_l4proto        = ipv6_get_l4proto,
#if IS_ENABLED(CONFIG_NF_CT_NETLINK)
    .tuple_to_nlattr    = ipv6_tuple_to_nlattr,
    .nlattr_tuple_size  = ipv6_nlattr_tuple_size,
    .nlattr_to_tuple    = ipv6_nlattr_to_tuple,
    .nla_policy     = ipv6_nla_policy,
#endif
    .me         = THIS_MODULE,
};

struct nf_conntrack_l4proto nf_conntrack_l4proto_tcp4 __read_mostly =
{
    .l3proto        = PF_INET,
    .l4proto        = IPPROTO_TCP,
    .name           = "tcp",
    .pkt_to_tuple       = tcp_pkt_to_tuple,
    .invert_tuple       = tcp_invert_tuple,
    .print_tuple        = tcp_print_tuple,
    .print_conntrack    = tcp_print_conntrack,
    .packet         = tcp_packet,
    .get_timeouts       = tcp_get_timeouts,
    .new            = tcp_new,
    .error          = tcp_error,
#if IS_ENABLED(CONFIG_NF_CT_NETLINK)
    .to_nlattr      = tcp_to_nlattr,
    .nlattr_size        = tcp_nlattr_size,
    .from_nlattr        = nlattr_to_tcp,
    .tuple_to_nlattr    = nf_ct_port_tuple_to_nlattr,
    .nlattr_to_tuple    = nf_ct_port_nlattr_to_tuple,
    .nlattr_tuple_size  = tcp_nlattr_tuple_size,
    .nla_policy     = nf_ct_port_nla_policy,
#endif
#if IS_ENABLED(CONFIG_NF_CT_NETLINK_TIMEOUT)
+---  8 lines: .ctnl_timeout  = {--------------------------------------------------------------------------------------------------------------------------------------
#endif /* CONFIG_NF_CT_NETLINK_TIMEOUT */
    .init_net       = tcp_init_net,
    .get_net_proto      = tcp_get_net_proto,
};

struct nf_conntrack_l4proto nf_conntrack_l4proto_udp4 __read_mostly =
{
    .l3proto        = PF_INET,
    .l4proto        = IPPROTO_UDP,
    .name           = "udp",
    .pkt_to_tuple       = udp_pkt_to_tuple,
    .invert_tuple       = udp_invert_tuple,
    .print_tuple        = udp_print_tuple,
    .packet         = udp_packet,
    .get_timeouts       = udp_get_timeouts,
    .new            = udp_new,
    .error          = udp_error,
#if IS_ENABLED(CONFIG_NF_CT_NETLINK)
    .tuple_to_nlattr    = nf_ct_port_tuple_to_nlattr,
    .nlattr_to_tuple    = nf_ct_port_nlattr_to_tuple,
    .nlattr_tuple_size  = nf_ct_port_nlattr_tuple_size,
    .nla_policy     = nf_ct_port_nla_policy,
#endif
#if IS_ENABLED(CONFIG_NF_CT_NETLINK_TIMEOUT)
+---  7 lines: .ctnl_timeout  = {--------------------------------------------------------------------------------------------------------------------------------------
#endif /* CONFIG_NF_CT_NETLINK_TIMEOUT */
    .init_net       = udp_init_net,
    .get_net_proto      = udp_get_net_proto,
};

struct nf_conntrack_l4proto nf_conntrack_l4proto_icmp __read_mostly =
{
    .l3proto        = PF_INET,
    .l4proto        = IPPROTO_ICMP,
    .name           = "icmp",
    .pkt_to_tuple       = icmp_pkt_to_tuple,
    .invert_tuple       = icmp_invert_tuple,
    .print_tuple        = icmp_print_tuple,
    .packet         = icmp_packet,
    .get_timeouts       = icmp_get_timeouts,
    .new            = icmp_new,
    .error          = icmp_error,
    .destroy        = NULL,
    .me         = NULL,
#if IS_ENABLED(CONFIG_NF_CT_NETLINK)
    .tuple_to_nlattr    = icmp_tuple_to_nlattr,
    .nlattr_tuple_size  = icmp_nlattr_tuple_size,
    .nlattr_to_tuple    = icmp_nlattr_to_tuple,
    .nla_policy     = icmp_nla_policy,
#endif
#if IS_ENABLED(CONFIG_NF_CT_NETLINK_TIMEOUT)
+---  7 lines: .ctnl_timeout  = {--------------------------------------------------------------------------------------------------------------------------------------
#endif /* CONFIG_NF_CT_NETLINK_TIMEOUT */
    .init_net       = icmp_init_net,
    .get_net_proto      = icmp_get_net_proto,
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
# IP Layer Flow

## Receive Direction
** IP layer receving packets begins from ip_rcv, ip_rcv will check the packet whether this packet is for this PC or other host. If it's for this PC, we will call ip_local_deliver, 
otherwise, we will call ip_forwad to forward this packet.**

```c
int ip_rcv(struct sk_buff *skb, struct net_device *dev, struct packet_type *pt, struct net_device *orig_dev)
{
	return NF_HOOK(NFPROTO_IPV4, NF_INET_PRE_ROUTING,
		       net, NULL, skb, dev, NULL,
		       ip_rcv_finish);	
}

static int ip_rcv_finish(struct net *net, struct sock *sk, struct sk_buff *skb)
{
	if (!skb_valid_dst(skb)) {
		int err = ip_route_input_noref(skb, iph->daddr, iph->saddr,
					       iph->tos, skb->dev);
		if (unlikely(err)) {
			if (err == -EXDEV)
				NET_INC_STATS_BH(net, LINUX_MIB_IPRPFILTER);
			goto drop;
		}
	}
	//Note: Here we may call different functions (ip_forward/ip_local_deliver) according to different destination.
	return dst_input(skb);	
}

int ip_route_input_noref(struct sk_buff *skb, __be32 daddr, __be32 saddr,
			 u8 tos, struct net_device *dev)
{
	res = ip_route_input_slow(skb, daddr, saddr, tos, dev);
}

static int ip_route_input_slow(struct sk_buff *skb, __be32 daddr, __be32 saddr,
			       u8 tos, struct net_device *dev)
{
	if (res.type == RTN_LOCAL) {
		err = fib_validate_source(skb, saddr, daddr, tos,
					  0, dev, in_dev, &itag);
		if (err < 0)
			goto martian_source;
		goto local_input;
	}
	# Note: Only non-local packet will enter this point, it means we need to forward this packet.
	err = ip_mkroute_input(skb, &res, &fl4, in_dev, daddr, saddr, tos);
out:	return err;

	
local_input:
	// Here, the dst.input = ip_local_deliver
	rth = rt_dst_alloc(net->loopback_dev, flags | RTCF_LOCAL, res.type,
			   IN_DEV_CONF_GET(in_dev, NOPOLICY), false, do_cache);
	skb_dst_set(skb, &rth->dst);
	
}
static int ip_mkroute_input(struct sk_buff *skb,
			    struct fib_result *res,
			    const struct flowi4 *fl4,
			    struct in_device *in_dev,
			    __be32 daddr, __be32 saddr, u32 tos)
{
	/* create a routing cache entry */
	return __mkroute_input(skb, res, in_dev, daddr, saddr, tos);
}

static int __mkroute_input(struct sk_buff *skb,
			   const struct fib_result *res,
			   struct in_device *in_dev,
			   __be32 daddr, __be32 saddr, u32 tos)
{
	rth = rt_dst_alloc(out_dev->dev, 0, res->type,
			   IN_DEV_CONF_GET(in_dev, NOPOLICY),
			   IN_DEV_CONF_GET(out_dev, NOXFRM), do_cache);
	rth->dst.input = ip_forward;
	skb_dst_set(skb, &rth->dst);
	
}

static struct rtable *rt_dst_alloc(struct net_device *dev,
				   unsigned int flags, u16 type,
				   bool nopolicy, bool noxfrm, bool will_cache)
{
		rt->dst.output = ip_output;
		if (flags & RTCF_LOCAL)
			rt->dst.input = ip_local_deliver;
}
```
## Send Direction


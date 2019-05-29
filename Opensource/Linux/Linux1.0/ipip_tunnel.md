# IPIP Tunnel

## Basic Introduction
IPIP tunnel is one tunnel technology. 
![ipip_introduction](./pic/ipip_tunnel_intro.png)

## Code Flow
![ipip_flow](./pic/ipip_flow.png)
```c
static int __init tunnel4_init(void)                                                                                                                         
{   
    if (inet_add_protocol(&tunnel4_protocol, IPPROTO_IPIP)) {
        printk(KERN_ERR "tunnel4 init: can't add protocol\n");                                                                                               
        return -EAGAIN;                                                                                                                                      
    }
#if defined(CONFIG_IPV6) || defined(CONFIG_IPV6_MODULE)
    if (inet_add_protocol(&tunnel64_protocol, IPPROTO_IPV6)) {
        printk(KERN_ERR "tunnel64 init: can't add protocol\n");                                                                                              
        inet_del_protocol(&tunnel4_protocol, IPPROTO_IPIP);                                                                                                  
        return -EAGAIN;                                                                                                                                      
    }
#endif
    return 0;                                                                                                                                                
}     

static const struct net_protocol tunnel4_protocol = {
    .handler    =   tunnel4_rcv,
    .err_handler    =   tunnel4_err,
    .no_policy  =   1,
    .netns_ok   =   1,
};

static int tunnel4_rcv(struct sk_buff *skb)
{
    struct xfrm_tunnel *handler;

    if (!pskb_may_pull(skb, sizeof(struct iphdr)))
        goto drop;

    for (handler = tunnel4_handlers; handler; handler = handler->next)
        if (!handler->handler(skb))
            return 0;

    icmp_send(skb, ICMP_DEST_UNREACH, ICMP_PORT_UNREACH, 0);

drop:
    kfree_skb(skb);
    return 0;
}

int xfrm4_tunnel_register(struct xfrm_tunnel *handler, unsigned short family)
{
    struct xfrm_tunnel **pprev;
    int ret = -EEXIST;
    int priority = handler->priority;
    
    mutex_lock(&tunnel4_mutex);
    
    for (pprev = fam_handlers(family); *pprev; pprev = &(*pprev)->next) {
        if ((*pprev)->priority > priority)
            break;
        if ((*pprev)->priority == priority)
            goto err;
    }
    
    handler->next = *pprev;
    *pprev = handler;
    
    ret = 0;

err:
    mutex_unlock(&tunnel4_mutex);
    
    return ret;
}

static int __init ipip_init(void)
{
    int err;

    printk(banner);

    err = register_pernet_device(&ipip_net_ops);
    if (err < 0)
        return err;
    err = xfrm4_tunnel_register(&ipip_handler, AF_INET);
    if (err < 0) {
        unregister_pernet_device(&ipip_net_ops);
        printk(KERN_INFO "ipip init: can't register tunnel\n");
    }
    return err;
}

static int ipip_rcv(struct sk_buff *skb)
{
    struct ip_tunnel *tunnel;
    const struct iphdr *iph = ip_hdr(skb);

    rcu_read_lock();
    if ((tunnel = ipip_tunnel_lookup(dev_net(skb->dev),
                    iph->saddr, iph->daddr)) != NULL) {
        if (!xfrm4_policy_check(NULL, XFRM_POLICY_IN, skb)) {
            rcu_read_unlock();
            kfree_skb(skb);
            return 0;
        }

        secpath_reset(skb);

        skb->mac_header = skb->network_header;
        skb_reset_network_header(skb);
        skb->protocol = htons(ETH_P_IP);
        skb->pkt_type = PACKET_HOST;

        tunnel->dev->stats.rx_packets++;
        tunnel->dev->stats.rx_bytes += skb->len;
        skb->dev = tunnel->dev;
        skb_dst_drop(skb);
        nf_reset(skb);
        ipip_ecn_decapsulate(iph, skb);
        netif_rx(skb);                                                                                                                                                                       
        rcu_read_unlock();
        return 0;
    }
    rcu_read_unlock();

    return -1;
}

      
```

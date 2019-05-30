# IPGRE tunnel

## IP GRE basic introduction

Ip gre tunnel is similar with ipip tunnel. Ip gre tunnel is one cicso recommended tunnel method.

## Code Flow
![ipgre_flow](./pic/ipgre_flow.png)

```c
static int ipgre_rcv(struct sk_buff *skb)
{
	skb->mac_header = skb->network_header;
	__pskb_pull(skb, offset);
    skb->dev = tunnel->dev; 
	skb_reset_network_header(skb);
	netif_rx(skb);
}

static netdev_tx_t ipgre_tunnel_xmit(struct sk_buff *skb, struct net_device *dev)
{
    iph             =   ip_hdr(skb);
    iph->version        =   4;
    iph->ihl        =   sizeof(struct iphdr) >> 2;
    iph->frag_off       =   df;
    iph->protocol       =   IPPROTO_GRE;
    iph->tos        =   ipgre_ecn_encapsulate(tos, old_iph, skb);
    iph->daddr      =   rt->rt_dst;
    iph->saddr      =   rt->rt_src;

    IPTUNNEL_XMIT();
    return NETDEV_TX_OK;
}
#define IPTUNNEL_XMIT() do {                        \
    int err;                            \
    int pkt_len = skb->len - skb_transport_offset(skb);     \
                                    \
    skb->ip_summed = CHECKSUM_NONE;                 \
    ip_select_ident(iph, &rt->u.dst, NULL);             \
                                    \
    err = ip_local_out(skb);                    \
    if (likely(net_xmit_eval(err) == 0)) {              \
        txq->tx_bytes += pkt_len;               \
        txq->tx_packets++;                  \
    } else {                            \
        stats->tx_errors++;                 \
        stats->tx_aborted_errors++;             \
    }                               \
} while (0)

int __ip_local_out(struct sk_buff *skb)
{                                                                                                                                                                                            
    struct iphdr *iph = ip_hdr(skb);

    iph->tot_len = htons(skb->len);
    ip_send_check(iph);
    return nf_hook(PF_INET, NF_INET_LOCAL_OUT, skb, NULL, skb_dst(skb)->dev,
               dst_output);
}
```
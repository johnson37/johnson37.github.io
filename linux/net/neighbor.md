# Neighbor System

## Basic Introduction 
**Neighbor system exist on Lay2.5, when Layer3 router system find the destination IP's route port, Neighbor 
will check the nexthop from router system, and get the nexthop's MAC address, usually we need to send ARP**

- ipv4: arp
- ipv6: nd

## Code Flow
**We will start from Layer 3's end ip_output**
```c
int ip_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
    struct net_device *dev = skb_dst(skb)->dev;

    IP_UPD_PO_STATS(net, IPSTATS_MIB_OUT, skb->len);

    skb->dev = dev;
    skb->protocol = htons(ETH_P_IP);

    return NF_HOOK_COND(NFPROTO_IPV4, NF_INET_POST_ROUTING,
                net, sk, skb, NULL, dev,
                ip_finish_output,
                !(IPCB(skb)->flags & IPSKB_REROUTED));
}

static int ip_finish_output(struct net *net, struct sock *sk, struct sk_buff *skb)
{
    return ip_finish_output2(net, sk, skb);
}

//Here, we need to find nexthop, and maybe create neighbor entry, add the MAC.
static int ip_finish_output2(struct net *net, struct sock *sk, struct sk_buff *skb)
{
    nexthop = (__force u32) rt_nexthop(rt, ip_hdr(skb)->daddr);
    neigh = __ipv4_neigh_lookup_noref(dev, nexthop);
    if (unlikely(!neigh))
        neigh = __neigh_create(&arp_tbl, &nexthop, dev, false);
    if (!IS_ERR(neigh)) {
        int res = dst_neigh_output(dst, neigh, skb);

        rcu_read_unlock_bh();
        return res;
    }
}
int neigh_resolve_output(struct neighbour *neigh, struct sk_buff *skb)
{
    int rc = 0;

    if (!neigh_event_send(neigh, skb)) {
        int err;
        struct net_device *dev = neigh->dev;
        unsigned int seq;

        if (dev->header_ops->cache && !neigh->hh.hh_len)
            neigh_hh_init(neigh);

        do {
            __skb_pull(skb, skb_network_offset(skb));
            seq = read_seqbegin(&neigh->ha_lock);
			//The first time, we cannot fill the da mac successfully, will free the skb.
            err = dev_hard_header(skb, dev, ntohs(skb->protocol),
                          neigh->ha, NULL, skb->len);
        } while (read_seqretry(&neigh->ha_lock, seq));

        if (err >= 0)
            rc = dev_queue_xmit(skb);
        else
            goto out_kfree_skb;
    }
out_kfree_skb:
    rc = -EINVAL;
    kfree_skb(skb);
    goto out;

}
//Note, dev_hard_header means eth_header. The first time we try , the neighbor's ha is null.
int eth_header(struct sk_buff *skb, struct net_device *dev,
           unsigned short type,
           const void *daddr, const void *saddr, unsigned int len)
{

    if (daddr) {
        memcpy(eth->h_dest, daddr, ETH_ALEN);
        return ETH_HLEN;
    }
    return -ETH_HLEN;
}
int __neigh_event_send(struct neighbour *neigh, struct sk_buff *skb)
{
    if (!(neigh->nud_state & (NUD_STALE | NUD_INCOMPLETE))) {
        if (NEIGH_VAR(neigh->parms, MCAST_PROBES) +
            NEIGH_VAR(neigh->parms, APP_PROBES)) {
            unsigned long next, now = jiffies;

            atomic_set(&neigh->probes,
                   NEIGH_VAR(neigh->parms, UCAST_PROBES));
            neigh->nud_state     = NUD_INCOMPLETE;
            neigh->updated = now;
            next = now + max(NEIGH_VAR(neigh->parms, RETRANS_TIME),
                     HZ/2);
            neigh_add_timer(neigh, next);
            immediate_probe = true;
        }
	}
	    if (neigh->nud_state == NUD_INCOMPLETE) {
        if (skb) {
            while (neigh->arp_queue_len_bytes + skb->truesize >
                   NEIGH_VAR(neigh->parms, QUEUE_LEN_BYTES)) {
                struct sk_buff *buff;

                buff = __skb_dequeue(&neigh->arp_queue);
                if (!buff)
                    break;
                neigh->arp_queue_len_bytes -= buff->truesize;
                kfree_skb(buff);
                NEIGH_CACHE_STAT_INC(neigh->tbl, unres_discards);
            }
            skb_dst_force(skb);
            __skb_queue_tail(&neigh->arp_queue, skb);
            neigh->arp_queue_len_bytes += skb->truesize;
        }
        rc = 1;
    }
out_unlock_bh:
    if (immediate_probe)
        neigh_probe(neigh);

}

static void neigh_probe(struct neighbour *neigh)
    __releases(neigh->lock)
{
    struct sk_buff *skb = skb_peek_tail(&neigh->arp_queue);
    /* keep skb alive even if arp_queue overflows */
    if (skb)
        skb = skb_clone(skb, GFP_ATOMIC);
    write_unlock(&neigh->lock);
    if (neigh->ops->solicit)
        neigh->ops->solicit(neigh, skb);
    atomic_inc(&neigh->probes);
    kfree_skb(skb);
}

static void arp_solicit(struct neighbour *neigh, struct sk_buff *skb)
{
    arp_send_dst(ARPOP_REQUEST, ETH_P_ARP, target, dev, saddr,
             dst_hw, dev->dev_addr, NULL, dst);

}

//Now, we have already send the arp request to nexthop, we expect to receive his response.
static int arp_rcv(struct sk_buff *skb, struct net_device *dev,
           struct packet_type *pt, struct net_device *orig_dev)
{
    return NF_HOOK(NFPROTO_ARP, NF_ARP_IN,
               dev_net(dev), NULL, skb, dev, NULL,
               arp_process);

}
//Here, we update the neighbor entry with the correct da mac.
static int arp_process(struct net *net, struct sock *sk, struct sk_buff *skb)
{
    neigh_update(n, sha, state,
             override ? NEIGH_UPDATE_F_OVERRIDE : 0);
	
}
int neigh_update(struct neighbour *neigh, const u8 *lladdr, u8 new,
		 u32 flags)
{
	if (new & NUD_CONNECTED)
		neigh_connect(neigh);
	else
		neigh_suspect(neigh);
	if (!(old & NUD_VALID)) {
		struct sk_buff *skb;

		/* Again: avoid dead loop if something went wrong */

		while (neigh->nud_state & NUD_VALID &&
		       (skb = __skb_dequeue(&neigh->arp_queue)) != NULL) {
			struct dst_entry *dst = skb_dst(skb);
			struct neighbour *n2, *n1 = neigh;
			write_unlock_bh(&neigh->lock);

			rcu_read_lock();

			/* Why not just use 'neigh' as-is?  The problem is that
			 * things such as shaper, eql, and sch_teql can end up
			 * using alternative, different, neigh objects to output
			 * the packet in the output path.  So what we need to do
			 * here is re-lookup the top-level neigh in the path so
			 * we can reinject the packet there.
			 */
			n2 = NULL;
			if (dst) {
				n2 = dst_neigh_lookup_skb(dst, skb);
				if (n2)
					n1 = n2;
			}
			n1->output(n1, skb);
			if (n2)
				neigh_release(n2);
			rcu_read_unlock();

			write_lock_bh(&neigh->lock);
		}
		__skb_queue_purge(&neigh->arp_queue);
		neigh->arp_queue_len_bytes = 0;
	}	
}

static void neigh_connect(struct neighbour *neigh)
{
    neigh_dbg(2, "neigh %p is connected\n", neigh);

    neigh->output = neigh->ops->connected_output;
}

int neigh_connected_output(struct neighbour *neigh, struct sk_buff *skb)
{
    struct net_device *dev = neigh->dev;
    unsigned int seq;
    int err;

    do {
        __skb_pull(skb, skb_network_offset(skb));
        seq = read_seqbegin(&neigh->ha_lock);
        err = dev_hard_header(skb, dev, ntohs(skb->protocol),
                      neigh->ha, NULL, skb->len);
    } while (read_seqretry(&neigh->ha_lock, seq));

    if (err >= 0)
        err = dev_queue_xmit(skb);
    else {
        err = -EINVAL;
        kfree_skb(skb);
    }
    return err;
}

``` 

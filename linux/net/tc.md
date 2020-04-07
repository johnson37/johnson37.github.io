# tc

## Filter

### set Filter

### Use Filter
```c
static inline struct sk_buff *handle_ing(struct sk_buff *skb,
                     struct packet_type **pt_prev,
                     int *ret, struct net_device *orig_dev)
{
#ifdef CONFIG_NET_CLS_ACT
    struct tcf_proto *cl = rcu_dereference_bh(skb->dev->ingress_cl_list);
    struct tcf_result cl_res;

+---  5 lines: If there's at least one ingress present somewhere (so-------------------------------------------------------------------------------------------------------------------------------------------
    if (!cl)
        return skb;
+---  4 lines: if (*pt_prev) {---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

    qdisc_skb_cb(skb)->pkt_len = skb->len;
    skb->tc_verd = SET_TC_AT(skb->tc_verd, AT_INGRESS);
    qdisc_bstats_cpu_update(cl->q, skb);

    switch (tc_classify(skb, cl, &cl_res, false)) {
    case TC_ACT_OK:
    case TC_ACT_RECLASSIFY:
        skb->tc_index = TC_H_MIN(cl_res.classid);
        break;
    case TC_ACT_SHOT:
        qdisc_qstats_cpu_drop(cl->q);
    case TC_ACT_STOLEN:
    case TC_ACT_QUEUED:
        kfree_skb(skb);
        return NULL;
    case TC_ACT_REDIRECT:
        /* skb_mac_header check was done by cls/act_bpf, so
         * we can safely push the L2 header back before
         * redirecting to another netdev
         */
        __skb_push(skb, skb->mac_len);
        skb_do_redirect(skb);
        return NULL;
    default:
        break;
    }
#endif /* CONFIG_NET_CLS_ACT */
    return skb;
}


int tc_classify(struct sk_buff *skb, const struct tcf_proto *tp,
        struct tcf_result *res, bool compat_mode)
{
    __be16 protocol = tc_skb_protocol(skb);
#ifdef CONFIG_NET_CLS_ACT
    const struct tcf_proto *old_tp = tp;
    int limit = 0;
    
reclassify:
#endif  
    for (; tp; tp = rcu_dereference_bh(tp->next)) {
        int err;

        if (tp->protocol != protocol &&
            tp->protocol != htons(ETH_P_ALL))
            continue;

        err = tp->classify(skb, tp, res);
#ifdef CONFIG_NET_CLS_ACT
        if (unlikely(err == TC_ACT_RECLASSIFY && !compat_mode))
            goto reset;
#endif
        if (err >= 0)
            return err;
    }

    return -1;
#ifdef CONFIG_NET_CLS_ACT
reset:
    if (unlikely(limit++ >= MAX_REC_LOOP)) {
        net_notice_ratelimited("%s: reclassify loop, rule prio %u, protocol %02x\n",
                       tp->q->ops->id, tp->prio & 0xffff,
                       ntohs(tp->protocol));
        return TC_ACT_SHOT;
    }

    tp = old_tp;
    protocol = tc_skb_protocol(skb);
    goto reclassify;
#endif
}
```
## Qdisc

### Register Qdisc
```c
static struct Qdisc_ops *qdisc_base;

int register_qdisc(struct Qdisc_ops *qops)
{
    struct Qdisc_ops *q, **qp;
    int rc = -EEXIST;

    write_lock(&qdisc_mod_lock); 
    for (qp = &qdisc_base; (q = *qp) != NULL; qp = &q->next)
        if (!strcmp(qops->id, q->id))
            goto out;
    qops->next = NULL;
    *qp = qops;
    rc = 0;
}
```
### Create Qdisc
```c
static struct Qdisc *
qdisc_create(struct net_device *dev, struct netdev_queue *dev_queue,
         struct Qdisc *p, u32 parent, u32 handle,
         struct nlattr **tca, int *errp)
{
    sch = qdisc_alloc(dev_queue, ops);
	
}

struct Qdisc *qdisc_alloc(struct netdev_queue *dev_queue,
              const struct Qdisc_ops *ops)
{
    sch->ops = ops;
    sch->enqueue = ops->enqueue;
    sch->dequeue = ops->dequeue;
    sch->dev_queue = dev_queue;    
}
```


### Use 

```
static inline int __dev_xmit_skb(struct sk_buff *skb, struct Qdisc *q,
                 struct net_device *dev,
                 struct netdev_queue *txq)
{
    spinlock_t *root_lock = qdisc_lock(q);
    bool contended;
    int rc;

    qdisc_pkt_len_init(skb);
    qdisc_calculate_pkt_len(skb, q);
+---  6 lines: Heuristic to force contended enqueues to serialize on a-----------------------------------------------------------------------------------------------------------------------------------------
    contended = qdisc_is_running(q);
    if (unlikely(contended))
        spin_lock(&q->busylock);
    
    spin_lock(root_lock);
+---  4 lines: if (unlikely(test_bit(__QDISC_STATE_DEACTIVATED, &q->state))) {---------------------------------------------------------------------------------------------------------------------------------
           qdisc_run_begin(q)) {
+----  5 lines: This is a work-conserving queue; there are no old skbs-----------------------------------------------------------------------------------------------------------------------------------------
    
        qdisc_bstats_update(q, skb);

+----  7 lines: if (sch_direct_xmit(skb, q, dev, txq, root_lock, true)) {--------------------------------------------------------------------------------------------------------------------------------------
            qdisc_run_end(q);

        rc = NET_XMIT_SUCCESS;
    } else {
        rc = q->enqueue(skb, q) & NET_XMIT_MASK;
+----  7 lines: if (qdisc_run_begin(q)) {----------------------------------------------------------------------------------------------------------------------------------------------------------------------
    }
    spin_unlock(root_lock);
    if (unlikely(contended))
        spin_unlock(&q->busylock);
    return rc;
}

```
### netem

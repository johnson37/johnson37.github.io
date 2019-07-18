# netif_rx
## Basic Introduction
Netif_rx是网络驱动中的一个常见函数，从netif_rx到netif_receive_skb可以视为网络驱动的收包终点，也是网络协议栈收包的起点。
本文主要总结从netif_rx到netif_receive_skb的过程。现在有些网络驱动依然沿用netif_rx, 有些则已经开始直接调用netif_receive_skb.
当然，直接使用netif_receive_skb 本质上网络驱动自己实现了从netif_rx到netif_receive_skb的过程。

## From netif_rx to netif_receive_skb
网络驱动中，收包时会有中断发生，中断发生后网络驱动读取数据，处理中断，并调用netif_rx/netif_receive_skb 要求协议栈处理报文。
从系统设计的角度考虑，中断中处理时间越短约好，这样中断尽早结束可以更及时的相应下一次中断。所以整个设计中，中断分成两部分，上半部和下半部。
中断的上半部，为网络驱动发生中断，进入中断处理函数读取报文，递交给netif_rx, 退出中断。中断的下半部指的是netif_rx到netif_receive_skb，
这一部分是通过软中断来实现的，并且是可以被打断的。

## Implementation
```c
int netif_rx(struct sk_buff *skb)
{
    unsigned int qtail;
    ret = enqueue_to_backlog(skb, get_cpu(), &qtail);
    put_cpu();
}
// Here, all skb is saved in input_pkt_queue.
static int enqueue_to_backlog(struct sk_buff *skb, int cpu,
                  unsigned int *qtail)
{
    struct softnet_data *sd;
    unsigned long flags;

    sd = &per_cpu(softnet_data, cpu);

    local_irq_save(flags);

    rps_lock(sd);
    if (skb_queue_len(&sd->input_pkt_queue) <= netdev_max_backlog) {
        if (skb_queue_len(&sd->input_pkt_queue)) {
enqueue:
            __skb_queue_tail(&sd->input_pkt_queue, skb);
            input_queue_tail_incr_save(sd, qtail);
            rps_unlock(sd);
            local_irq_restore(flags);
            return NET_RX_SUCCESS;
        }

        /* Schedule NAPI for backlog device
         * We can use non atomic operation since we own the queue lock
         */
        if (!__test_and_set_bit(NAPI_STATE_SCHED, &sd->backlog.state)) {
            if (!rps_ipi_queued(sd))
                ____napi_schedule(sd, &sd->backlog);
        }
        goto enqueue;
    }

    sd->dropped++;
    rps_unlock(sd);

    local_irq_restore(flags);

    kfree_skb(skb);
    return NET_RX_DROP;
}
//Here, we add sd->backlog --> sd->poll_list, and trigger RX_SOFTIRQ softirq
static inline void ____napi_schedule(struct softnet_data *sd,
                     struct napi_struct *napi)
{
    list_add_tail(&napi->poll_list, &sd->poll_list);
    __raise_softirq_irqoff(NET_RX_SOFTIRQ);
}

//Here n->poll means sd->backlog->poll == process_backlog
static void net_rx_action(struct softirq_action *h)
{
    struct softnet_data *sd = &__get_cpu_var(softnet_data);
    unsigned long time_limit = jiffies + 2;
    int budget = netdev_budget;
    void *have;

    local_irq_disable();

    while (!list_empty(&sd->poll_list)) {
        struct napi_struct *n;
        int work, weight;
	...
        work = 0;
        if (test_bit(NAPI_STATE_SCHED, &n->state)) {
            work = n->poll(n, weight);
            trace_napi_poll(n);
        }
        budget -= work;
	}
}
static int __init net_dev_init(void)
{
    for_each_possible_cpu(i) {
		sd->backlog.poll = process_backlog;
	}
}	
static int process_backlog(struct napi_struct *napi, int quota)
{
        while ((skb = __skb_dequeue(&sd->process_queue))) {
            local_irq_enable();
            __netif_receive_skb(skb);
            local_irq_disable();
            input_queue_head_incr(sd);
            if (++work >= quota) {
                local_irq_enable();
                return work;
            }
        }
        qlen = skb_queue_len(&sd->input_pkt_queue);
        if (qlen)
            skb_queue_splice_tail_init(&sd->input_pkt_queue,
                           &sd->process_queue);

}	
```
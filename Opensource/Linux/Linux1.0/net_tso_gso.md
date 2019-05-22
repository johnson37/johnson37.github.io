# Net TSO & GSO

## Concept
- TSO: TCP Segmentation Offload.(是一种利用网卡来对大数据包进行自动分段，降低CPU负载的技术。 其主要是延迟分段。)
- GSO: Generic Segmentation Offload (GSO是协议栈是否推迟分段，在发送到网卡之前判断网卡是否支持TSO，如果网卡支持TSO则让网卡分段，否则协议栈分完段再交给驱动。 如果TSO开启，GSO会自动开启。)

## Code Flow

### Enable TSO in Enet Driver
**TSO is one hardware characteristic. Each net hardware device may support it or not. Once this net hardware device support TSO, we can enable the feature.**
**Take Intel e1000 as one example**

```c
static struct pci_driver e1000_driver = {
    .name     = e1000_driver_name,
    .id_table = e1000_pci_tbl,
    .probe    = e1000_probe,                                                                                                                                                                 
    .remove   = e1000_remove,
    .shutdown = e1000_shutdown,
    .err_handler = &e1000_err_handler
};

static int e1000_probe(struct pci_dev *pdev, const struct pci_device_id *ent)                                                                                                                
{
	if ((hw->mac_type >= e1000_82544) &&
       (hw->mac_type != e1000_82547))
        netdev->hw_features |= NETIF_F_TSO;

    netdev->priv_flags |= IFF_SUPP_NOFCS;

    netdev->features |= netdev->hw_features;
    netdev->hw_features |= (NETIF_F_RXCSUM |
                NETIF_F_RXALL |
                NETIF_F_RXFCS);

+---  4 lines: if (pci_using_dac) {----------------------------------------------------------------------------------------------------------------------------------------------------------
        
    netdev->vlan_features |= (NETIF_F_TSO |
                  NETIF_F_HW_CSUM |
                  NETIF_F_SG);
				  
	
	err = register_netdev(netdev);

}
```

### Enable GSO in register_netdevice

```c
#define NETIF_F_SOFT_FEATURES	(NETIF_F_GSO | NETIF_F_GRO)

int register_netdevice(struct net_device *dev)
{
	/* Transfer changeable features to wanted_features and enable
	 * software offloads (GSO and GRO).
	 */
	dev->hw_features |= NETIF_F_SOFT_FEATURES;
	dev->features |= NETIF_F_SOFT_FEATURES;
	dev->wanted_features = dev->features & dev->hw_features;
}
```

### TCP send msg , net stack will choose the mss value according to gso

```c
int tcp_sendmsg(struct sock *sk, struct msghdr *msg, size_t size)
{
	mss_now = tcp_send_mss(sk, &size_goal, flags);
}

static int tcp_send_mss(struct sock *sk, int *size_goal, int flags)
{
	int mss_now;

	mss_now = tcp_current_mss(sk);
	*size_goal = tcp_xmit_size_goal(sk, mss_now, !(flags & MSG_OOB));

	return mss_now;
}
static unsigned int tcp_xmit_size_goal(struct sock *sk, u32 mss_now,
				       int large_allowed)
{
	struct tcp_sock *tp = tcp_sk(sk);
	u32 new_size_goal, size_goal;

	if (!large_allowed || !sk_can_gso(sk))
		return mss_now;

	/* Note : tcp_tso_autosize() will eventually split this later */
	new_size_goal = sk->sk_gso_max_size - 1 - MAX_TCP_HEADER;
	new_size_goal = tcp_bound_to_half_wnd(tp, new_size_goal);

	/* We try hard to avoid divides here */
	size_goal = tp->gso_segs * mss_now;
	if (unlikely(new_size_goal < size_goal ||
		     new_size_goal >= size_goal + mss_now)) {
		tp->gso_segs = min_t(u16, new_size_goal / mss_now,
				     sk->sk_gso_max_segs);
		size_goal = tp->gso_segs * mss_now;
	}

	return max(size_goal, mss_now);
}
static inline bool net_gso_ok(netdev_features_t features, int gso_type)
{
	netdev_features_t feature = gso_type << NETIF_F_GSO_SHIFT;
	
	return (features & feature) == feature;
}
```
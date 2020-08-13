# ifconfig stat

When we use Linux ifconfig command, we will see some information about this Interface. This interface maybe is one Hw NIC , maybe is one virtual interface.

Also we can get these statistics from /proc/net/dev. For example, below is output from my person PC:

```c
[johnsonz@sfu03 core]$ cat /proc/net/dev
Inter-|   Receive                                                |  Transmit
 face |bytes    packets errs drop fifo frame compressed multicast|bytes    packets errs drop fifo colls carrier compressed
    lo: 262835134 2302390    0    0    0     0          0         0 262835134 2302390    0    0    0     0       0          0
  eth0: 358045654682 533079317    0    0    0     0          0   4916471 715119455714 709417293    0    0    0     0       0          0
  eth1: 1259560324 2133868    5    0    0     0          0         0 1038067921 1061794    0    0    0     0       0          0
virbr0:       0       0    0    0    0     0          0         0     1710       5    0    0    0     0       0          0
virbr0-nic:       0       0    0    0    0     0          0         0        0       0    0 734706    0     0       0          0
  eth4:   58212     625    0    0    0     0          0         0    63948     771    0    0    0     0       0          0

```

If this is one real HW Interface, HW provider will provide the net device' driver. In net device's driver, we will record these statistics. However, sometimes, we will met some virtual interface,
How can we get these statistics for this virtual Interface?

For example, below is one Bridge device's statistics.

```c
static int br_pass_frame_up(struct sk_buff *skb)
{
    struct net_device *indev, *brdev = BR_INPUT_SKB_CB(skb)->brdev;
    struct net_bridge *br = netdev_priv(brdev);
    struct net_bridge_vlan_group *vg;
    struct pcpu_sw_netstats *brstats = this_cpu_ptr(br->stats);

    u64_stats_update_begin(&brstats->syncp);
    brstats->rx_packets++;
    brstats->rx_bytes += skb->len;
    u64_stats_update_end(&brstats->syncp);
	...
}
```
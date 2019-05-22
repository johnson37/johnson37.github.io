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
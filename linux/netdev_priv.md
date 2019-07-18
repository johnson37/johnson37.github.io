# netdev_priv

## Basic Introduction
netdev_priv通常用在网络驱动当中，当网络设备通过alloc_netdev注册时，不同的网络厂家可能有一些私有数据，所以alloc时会额外申请
一块空间存放私有数据。netdev_priv能到读取到私有数据，并做特殊处理。

## Code
```c
#define alloc_netdev(sizeof_priv, name, name_assign_type, setup) \
    alloc_netdev_mqs(sizeof_priv, name, name_assign_type, setup, 1, 1)

struct net_device *alloc_netdev_mqs(int sizeof_priv, const char *name,
        unsigned char name_assign_type,
        void (*setup)(struct net_device *),
        unsigned int txqs, unsigned int rxqs)
{
    alloc_size = sizeof(struct net_device);
    if (sizeof_priv) {
        /* ensure 32-byte alignment of private area */
        alloc_size = ALIGN(alloc_size, NETDEV_ALIGN);
        alloc_size += sizeof_priv;
    }
    /* ensure 32-byte alignment of whole construct */
    alloc_size += NETDEV_ALIGN - 1;

    p = kzalloc(alloc_size, GFP_KERNEL | __GFP_NOWARN | __GFP_REPEAT);
    if (!p)
        p = vzalloc(alloc_size);
    if (!p)
        return NULL;
	...
}	
```
```c
static inline void *netdev_priv(const struct net_device *dev)
{
    return (char *)dev + ALIGN(sizeof(struct net_device), NETDEV_ALIGN);
}
```
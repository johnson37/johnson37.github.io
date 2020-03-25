# Notifier

## Basic Introduction
Notifier 是Linux 内核子模块之间的通讯机制。整个机制类似于“订阅-发布”机制，当某些子系统需要监听某些事件时，就订阅某个事件，当事件发生时，事件的发布者会通知每个订阅者。

## Usage
### 订阅者
- 需要事件的内核子系统作为订阅者，需要针对某个事件，在Linux中某个chain进行注册。与之相对，在不需要监听时，取消对事件的注册。
- 注册的时候会通过函数调用的方式，指明需要订阅的chain，以及事件发生后的回调函数。
- 注册函数： notifier_chain_register
- 取消注册：notifier_chain_unregister

```c
int notifier_chain_register(struct notifier_block **list, struct notifier_block *n)                                                                                         
{ 
    write_lock(&notifier_lock);
    while (*list) {            
        if (n->priority > (*list)->priority)
            break;                                                                                                                                                                                         
        list = &((*list)->next);    
    }
    n->next = *list;           
    *list = n;                 
    write_unlock(&notifier_lock);   
    return 0;                  
} 

int notifier_chain_unregister(struct notifier_block **nl, struct notifier_block *n)
{
    write_lock(&notifier_lock);
    while ((*nl) != NULL) {
        if ((*nl) == n) {
            *nl = n->next;
            write_unlock(&notifier_lock);
            return 0;
        }
        nl = &((*nl)->next);
    }
    write_unlock(&notifier_lock);
    return -ENOENT;
}

```

### 发布者
当相应的事件发生，发布者会逐个调用链表中订阅者的回调函数。
```c
int notifier_call_chain(struct notifier_block **n, unsigned long val, void *v)
{
    int ret = NOTIFY_DONE;
    struct notifier_block *nb = *n;

    while (nb) {
        ret = nb->notifier_call(nb, val, v);
        if (ret & NOTIFY_STOP_MASK) {
            return ret;
        }
        nb = nb->next;
    }
    return ret;
}

```

## Example

```c
struct notifier_block br_device_notifier = {                                                                                                                                                               
    br_device_event,
    NULL,
    0
};

static int br_device_event(struct notifier_block *unused, unsigned long event, void *ptr)
{
    struct net_device *dev;
    struct net_bridge_port *p;

    dev = ptr;
    p = dev->br_port;

    if (p == NULL)
        return NOTIFY_DONE;

    switch (event) {
    case NETDEV_CHANGEADDR:
        read_lock(&p->br->lock);
        br_fdb_changeaddr(p, dev->dev_addr);
        br_stp_recalculate_bridge_id(p->br);
        read_unlock(&p->br->lock);
        break;

    case NETDEV_GOING_DOWN:
        /* extend the protocol to send some kind of notification? */
        break;

    case NETDEV_DOWN:
        if (p->br->dev.flags & IFF_UP) {
            read_lock(&p->br->lock);
            br_stp_disable_port(dev->br_port);
            read_unlock(&p->br->lock);
        }
        break;
	}
}
static struct notifier_block *netdev_chain = NULL;
static int __init br_init(void)
{
    register_netdevice_notifier(&br_device_notifier);

    return 0;
}

int register_netdevice_notifier(struct notifier_block *nb)
{                                                                                                                                                                                                          
    return notifier_chain_register(&netdev_chain, nb);
}
static void __exit br_deinit(void)
{
    unregister_netdevice_notifier(&br_device_notifier);
}

int unregister_netdevice_notifier(struct notifier_block *nb)
{                                                                                                                                                                                                          
    return notifier_chain_unregister(&netdev_chain, nb);
}

```

现在注册的部分已经Ready，就等时间触发并调用了。

```c
int dev_change_flags(struct net_device *dev, unsigned flags)
{
	    if ((old_flags ^ flags)&IFF_UP) { /* Bit is different  ? */
        ret = ((old_flags & IFF_UP) ? dev_close : dev_open)(dev);
    }

}
int dev_close(struct net_device *dev)
{
	notifier_call_chain(&netdev_chain, NETDEV_DOWN, dev);
}
```

## Note
这是Notifier的简单实现和使用，上述代码取自Linux 2.4， 随着Linux版本的提升，Notifier的功能也变的复杂，出现了几类不同的Notifier。

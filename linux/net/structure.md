# Structure

主要是对Linux网络协议栈中的主要结构体做一个说明。

## sk_buff
**This structure comes from Linux 2.6 Version**

```c
struct sk_buff {
    /* These two members must be first. */
    struct sk_buff      *next;   //sk_buff 是由双向链表来管理的。
    struct sk_buff      *prev;

    ktime_t         tstamp;		// 记录一个skb packet的收包时间。

    struct sock     *sk;		// This is one variable which is related with socket, if the packet 
								// is send by local or received by local process, then we need this.
								// If we only to forward the packet, the pointer is NULL.
    struct net_device   *dev;	//If the packet is received, it means which device the packet is received from,
								// If the packet will be sent, it means which device the packet is sent through.

    /*
     * This is the control buffer. It is free to use for every
     * layer. Please put your private variables there. If you
     * want to keep them across layers you have to do a skb_clone()
     * first. This is owned by whoever has the skb queued ATM.
     */
    char            cb[48] __aligned(8);

    unsigned long       _skb_refdst;
#ifdef CONFIG_XFRM
    struct  sec_path    *sp;
#endif
    unsigned int        len,   // len 指的是当前协议包的数据长度，处在网络层，即IP报文的长度，处在TCP层，即TCP报文的长度。
                data_len;		// 只有分片报文才有效。
    __u16           mac_len,	// size of MAC Header.
                hdr_len;
    union {
        __wsum      csum;		// checksum
        struct {
            __u16   csum_start;
            __u16   csum_offset;
        };
    };
    __u32           priority;	//Qos related
    kmemcheck_bitfield_begin(flags1);
    __u8            local_df:1,
                cloned:1,		//This skb buffer is cloned from another skb.
                ip_summed:2,
                nohdr:1,
                nfctinfo:3;		//netfilter conntract related.
    __u8            pkt_type:3,	//Initialzed by eth_type_trans
                fclone:2,
                ipvs_property:1,
                peeked:1,
                nf_trace:1;
    kmemcheck_bitfield_end(flags1);
    __be16          protocol;

    void            (*destructor)(struct sk_buff *skb);	//It's one callback function. It ususlly is initialized by socket.
														//When skb is freed, it may be triggered to handle memory.
#if defined(CONFIG_NF_CONNTRACK) || defined(CONFIG_NF_CONNTRACK_MODULE)
    struct nf_conntrack *nfct;		//related with netfilter conntrack
    struct sk_buff      *nfct_reasm;//related with netfilter conntrack
#endif
#ifdef CONFIG_BRIDGE_NETFILTER
    struct nf_bridge_info   *nf_bridge;	// Netfilter related
#endif

    int         skb_iif;
#ifdef CONFIG_NET_SCHED
    __u16           tc_index;   /* traffic control index */	// Traffic control related
#ifdef CONFIG_NET_CLS_ACT
    __u16           tc_verd;    /* traffic control verdict */
#endif
#endif

    __u32           rxhash;

    kmemcheck_bitfield_begin(flags2);
    __u16           queue_mapping:16;
#ifdef CONFIG_IPV6_NDISC_NODETYPE
    __u8            ndisc_nodetype:2,
                deliver_no_wcard:1;
#else
    __u8            deliver_no_wcard:1;
#endif
    kmemcheck_bitfield_end(flags2);

    /* 0/14 bit hole */

#ifdef CONFIG_NET_DMA
    dma_cookie_t        dma_cookie;
#endif
#ifdef CONFIG_NETWORK_SECMARK
    __u32           secmark;
#endif
    union {
        __u32       mark;
        __u32       dropcount;
    };

    __u16           vlan_tci;

    sk_buff_data_t      transport_header;
    sk_buff_data_t      network_header;
    sk_buff_data_t      mac_header;
    /* These elements must be at the end, see alloc_skb() for details.  */
    sk_buff_data_t      tail;
    sk_buff_data_t      end;
    unsigned char       *head,
                *data;
    unsigned int        truesize;
    atomic_t        users;
};


```

### head/end/data/tail
![head/end/data/tail](./pic/skb_data.PNG)

skb->data, 是指当前有效数据的头
skb->tail，是指当前有效数据的尾


# TCP For Linux 1.0

## TCP Basic Knowledge
### TCP 3 handshake
- client 主动发起，发送SYNC
- server 被动发起，收到client的SYNC, 发送SYNC+ACK
- client 发送ACK，握手成功

![握手](./pic/handshake.png)

### TCP 4 times to wave

![挥手](./pic/wave.jpg)

- client 主动发起，发送FIN
- Server response ack
- server side send FIN
- client side response ack

## Basic Usage for TCP Socket on Application Layer

### Server Side
- socket: create one socket
- bind: bind socket with one port
- listen: set socket to listen state
- accept: receive clients' socket request
- recv: receive packets
- send: send packets

### Client Side
- socket: create one socket
- connect: connect remote server
- recv: receive packets
- send: send packets

## Code Flow

### Listen
**Before tcp client connect tcp server, tcp server need to bind, listen & accept**
**In Listen state,  we find tcp server 's socket state change from TCP_CLOSE to TCP_LISEN**

```c
static int 
sock_listen(int fd, int backlog)
{                                                                                                                                                                                              
    struct socket *sock;    
    
    DPRINTF((net_debug, "NET: sock_listen: fd = %d\n", fd));
    if (fd < 0 || fd >= NR_OPEN || current->filp[fd] == NULL)
        return (-EBADF);
    if (!(sock = sockfd_lookup(fd, NULL))) return (-ENOTSOCK);
    if (sock->state != SS_UNCONNECTED) {
        DPRINTF((net_debug, "NET: sock_listen: socket isn't unconnected\n"));
        return (-EINVAL);
    }   
    if (sock->ops && sock->ops->listen) sock->ops->listen(sock, backlog);
    sock->flags |= SO_ACCEPTCON;
    return (0);
}   

inet_listen(struct socket *sock, int backlog)
{
    struct sock *sk;

    sk = (struct sock *) sock->data;
    if (sk == NULL) {
        printk("Warning: sock->data = NULL: %d\n" , __LINE__);
        return (0);
    }

    /* We may need to bind the socket. */
    if (sk->num == 0) { 
        sk->num = get_new_socknum(sk->prot, 0);
        if (sk->num == 0) return (-EAGAIN);
        put_sock(sk->num, sk);
        sk->dummy_th.source = ntohs(sk->num);
    }

    /* We might as well re use these. */
    sk->max_ack_backlog = backlog;
    if (sk->state != TCP_LISTEN) {
        sk->ack_backlog = 0;
        sk->state = TCP_LISTEN;
    }
    return (0);                                                                                                                                                                                
}

```

### Connect
**In connect sys call, we will change sk->state from TCP_CLOSE to TCP_SYNC_SENT. we will wait until the state is not TCP_SYNC_SENT or TCP_SYNC_RECV**
```c
asmlinkage int
sys_socketcall(int call, unsigned long *args)
{
    case SYS_CONNECT:
        er = verify_area(VERIFY_READ, args, 3 * sizeof(long));
        if (er) 
            return er;
        return (sock_connect(get_fs_long(args + 0),
                             (struct sockaddr *)get_fs_long(args + 1),
                             get_fs_long(args + 2)));                         
}

static int 
inet_connect(struct socket *sock, struct sockaddr * uaddr,
             int addr_len, int flags) 
{
	err = sk->prot->connect(sk, (struct sockaddr_in *)uaddr, addr_len);
	while (sk->state == TCP_SYN_SENT || sk->state == TCP_SYN_RECV) {
        interruptible_sleep_on(sk->sleep);
        if (current->signal & ~current->blocked) {
            sti();
            return (-ERESTARTSYS);
        }
	}
}

//In tcp_connect, sk->state changes from TCP_CLOSE to TCP_CONNECT_SENT.
static int
tcp_connect(struct sock *sk, struct sockaddr_in *usin, int addr_len)
{
    sk->inuse = 1;
    sk->daddr = sin.sin_addr.s_addr;
    sk->write_seq = jiffies * SEQ_TICK - seq_offset;
    sk->window_seq = sk->write_seq;
    sk->rcv_ack_seq = sk->write_seq - 1;
    sk->err = 0;
    sk->dummy_th.dest = sin.sin_port;
    release_sock(sk);
	buff = sk->prot->wmalloc(sk, MAX_SYN_SIZE, 0, GFP_KERNEL);
	    sk->inuse = 1;
    buff->mem_addr = buff;
    buff->mem_len = MAX_SYN_SIZE;
    buff->len = 24;
    buff->sk = sk;
    buff->free = 1;
    t1 = (struct tcphdr *) buff->data;

    /* Put in the IP header and routing stuff. */
    /* We need to build the routing stuff fromt the things saved in skb. */
    tmp = sk->prot->build_header(buff, sk->saddr, sk->daddr, &dev,
                                 IPPROTO_TCP, NULL, MAX_SYN_SIZE, sk->ip_tos, sk->ip_ttl);
    t1->rst = 0;
    t1->urg = 0;
    t1->psh = 0;
    t1->syn = 1; //SYNC
    t1->urg_ptr = 0;
    t1->doff = 6;
	
    sk->state = TCP_SYN_SENT;
    sk->rtt = TCP_CONNECT_TIME;
    reset_timer(sk, TIME_WRITE, TCP_CONNECT_TIME);    /* Timer for repeating the SYN until an answer */
    sk->retransmits = TCP_RETR2 - TCP_SYN_RETRIES;

    sk->prot->queue_xmit(sk, dev, buff, 0);

    release_sock(sk);
    return (0);
}


```

**Now in server side, we receive SYNC, and then we need to response TCP_CLIENT SYNC+ACK packet, and server's state change to TCP_SYN_RECV **

```c
int
tcp_rcv(struct sk_buff *skb, struct device *dev, struct options *opt,                                                                                                                          
        unsigned long daddr, unsigned short len,
        unsigned long saddr, int redo, struct inet_protocol * protocol)
{
	    switch (sk->state) {
			case TCP_LISTEN:
				if (th->syn) {
					tcp_conn_request(sk, skb, daddr, saddr, opt, dev);
					release_sock(sk);
					return (0);
				}
		}
		
}

static void
tcp_conn_request(struct sock *sk, struct sk_buff *skb,
                 unsigned long daddr, unsigned long saddr,
                 struct options *opt, struct device *dev)
{
	tmp = sk->prot->build_header(buff, newsk->saddr, newsk->daddr, &dev,
                                 IPPROTO_TCP, NULL, MAX_SYN_SIZE, sk->ip_tos, sk->ip_ttl);

	newsk->state = TCP_SYN_RECV;

	t1->ack = 1;
	t1->syn = 1;
	
	newsk->prot->queue_xmit(newsk, dev, buff, 0);
}

```

**Now in client side, we receive the SYNC+ACK, and send ack to server side. Note that, after TCP_SYN_SENT, we don't get one break, so we will enter TCP_SYN_RECV, and at last TCP_ESTABLISHED**

```c
int
tcp_rcv(struct sk_buff *skb, struct device *dev, struct options *opt,                                                                                                                          
        unsigned long daddr, unsigned short len,
        unsigned long saddr, int redo, struct inet_protocol * protocol)
{
    case TCP_SYN_SENT:
        if (!tcp_ack(sk, th, saddr, len)) {
                tcp_reset(daddr, saddr, th,
                          sk->prot, opt, dev, sk->ip_tos, sk->ip_ttl);
                kfree_skb(skb, FREE_READ);
                release_sock(sk);
                return (0);
            }

            /*
             * If the syn bit is also set, switch to
             * tcp_syn_recv, and then to established.
             */
            if (!th->syn) {
                kfree_skb(skb, FREE_READ);
                release_sock(sk);
                return (0);
            }

            /* Ack the syn and fall through. */
            sk->acked_seq = th->seq + 1;
            sk->fin_seq = th->seq;
            tcp_send_ack(sk->sent_seq, th->seq + 3,
                         sk, th, sk->daddr);

        case TCP_SYN_RECV:
            if (!tcp_ack(sk, th, saddr, len)) {
                tcp_reset(daddr, saddr, th,
                          sk->prot, opt, dev, sk->ip_tos, sk->ip_ttl);
                kfree_skb(skb, FREE_READ);
                release_sock(sk);
                return (0);
            }
            sk->state = TCP_ESTABLISHED;
            tcp_options(sk, th);
            sk->dummy_th.dest = th->source;
            sk->copied_seq = sk->acked_seq - 1;
            if (!sk->dead) {
                sk->state_change(sk);
            }
}

static void
tcp_send_ack(unsigned long sequence, unsigned long ack,
             struct sock *sk,
             struct tcphdr *th, unsigned long daddr)
{
    buff = sk->prot->wmalloc(sk, MAX_ACK_SIZE, 1, GFP_ATOMIC);
    tmp = sk->prot->build_header(buff, sk->saddr, daddr, &dev,
                                 IPPROTO_TCP, sk->opt, MAX_ACK_SIZE, sk->ip_tos, sk->ip_ttl);
    t1->ack = 1;
    t1->res1 = 0;
    t1->res2 = 0;
    t1->rst = 0;
    t1->urg = 0;
    t1->syn = 0;
    t1->psh = 0;
    t1->fin = 0;
    sk->prot->queue_xmit(sk, dev, buff, 1);	
}

```

**In server side, we recieve the ack message , and change state from TCP_SYN_RECV to ESTABLISHED**
```c
int
tcp_rcv(struct sk_buff *skb, struct device *dev, struct options *opt,
        unsigned long daddr, unsigned short len,
        unsigned long saddr, int redo, struct inet_protocol * protocol)
{

        case TCP_SYN_RECV:
            if (!tcp_ack(sk, th, saddr, len)) {
                tcp_reset(daddr, saddr, th,
                          sk->prot, opt, dev, sk->ip_tos, sk->ip_ttl);
                kfree_skb(skb, FREE_READ);
                release_sock(sk);
                return (0);
            }
            sk->state = TCP_ESTABLISHED;
}
```

**Until here, both server side and client side enter ESTABLISH State**

# traceroute

traceroute的工作原理就是利用了ping机制，从ttl=1开始逐个递增，直接等到目标ip的reply。
如此途径的路由就会这个的回应ttl=0时对应的icmp消息。


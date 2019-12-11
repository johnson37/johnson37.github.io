# watchdog

https://www.ibm.com/developerworks/cn/linux/l-cn-watchdog/index.html

## Basic Introduction

Watchdog在实现上可以是硬件电路也可以是软件定时器，能够在系统出现故障时自动重新启动系统。
在Linux 内核下, watchdog的基本工作原理是：当watchdog启动后(即/dev/watchdog 设备被打开后)，
如果在某一设定的时间间隔内/dev/watchdog没有被执行写操作, 硬件watchdog电路或软件定时器就会重新启动系统。


## Implementation
从实现层面讲，watchdog分成应用层的watchdog daemon和内核层的驱动设备。

### WTD Driver
Path: /linux/drivers/watchdog
如果是硬件看门狗，通常情况下，方案提供商会给出自己设备的驱动程序，或者使用该目录下的软件看门狗。

WTD Driver 层会在启动初期，创建/dev/watchdog 字符设备。

### Application Daemon
Daemon 作用是定期的读写watchdog设备。

#### watchdog start
```c
int wdt_fd = -1;
wdt_fd = open("/dev/watchdog", O_WRONLY);
if (wdt_fd == -1)
{
    // fail to open watchdog device
}
```


#### Feed watchdog
至于具体的跟watchdog的交互内容，还是要跟对应的内核驱动相匹配。

```c
if (wdt_fd != -1)
    write(wdt_fd, "a", 1);
```

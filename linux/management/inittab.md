# Init

Linux init process will handle below steps:
- Read /etc/inittab file
- run /etc/init.d/rc.sysinit
- run /etc/init.d/rc
- run /etc/init.d/rc3.d //rc5.d

## Inittab
Usually, we set the runlevel in this file. Sometimes, we could define programs initializaion.

- the runlevel in which init will start the system by default
- programs init will run to initialize the system
- standard processes init will start for each runlevel
- scripts init will run to implement each runlevel


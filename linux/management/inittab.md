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

### inittab style

/etc/inittab文件格式：id:run-levels:action:process，共包含4项，用冒号分隔，其中某些部份可以为空，各项详细解释如下： 

- id: 标识符，一般为两位字母或数字，该标识符唯一，在配置文件中不能重复。
- run-levels: 
	-	0-halt
	-	1-Single User mode
	-	2-Multiuser,without NFS
	-	3-Full multiuser mode
	-	4-unused
	-	5-X11 具备网络功能的图形用户界面
	-	6-reboot 关闭所有运行的进程并重新启动系统
- action:
- process:
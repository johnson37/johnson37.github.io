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
**0-halt**
**1-Single User mode**
**2-Multiuser,without NFS**
**3-Full multiuser mode**
**4-unused**
**5-X11 具备网络功能的图形用户界面**
**6-reboot 关闭所有运行的进程并重新启动系统**
- action: 用于指定process进程相关的操作
**respawn: 一旦第4项的命令中止，重新运行该命令 **
**wait: 执行第4项的process，等待其结束并运行其他命令 **
**once: 执行第4项的process，不等待其结束即运行其他命令 **
**boot: 无论运行在哪个等级，都执行该命令 **
**bootwait: 无论运行在哪个等级，都执行该命令，并等待其结束 **
**off: 忽略该配置行 **
**ondemand: 进入oncommand执行等级时，执行第4项的命令 **
**initdefault: 系统启动后指定进入的执行等级，该行不需要指定process **
**sysinit: 无论在哪个执行等级，系统会在执行boot及bootwait之前执行第四项的process **
**powerwait: 系统供电不足时，执行指定的process，并等待其执行结束 **
**powerokwait: 系统供电恢复正常时，执行指定的process，并等待其执行结束 **
**powerfailnow: 系统供电严重不足时，执行指定的process **
**powerfail: 系统供电错误时，执行指定的process，不等待其结束 **
**ctrlaltdel: 当用户按下【Ctrl+Alt+Del】时执行第4项指定的 process。 **
**kbrequest: 当用户按下特殊的组合键时执行第4项指定的process，此组合键需在keymaps文件定义。**

- process: 指定要运行的shell command
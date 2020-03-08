# command analysis

## procps

### free
call sysinfo api

### kill
call kill api

### ps
read /proc/pid/xxx

### renice
call getpriority/setpriority api

### sysctl
modify /proc file, which is the environmental variables.

sysctl 通过proc文件实现用户态跟内核态的交互。

### pidof
找出进程对应的pid。类似于ps，逐个读取proc文件，查看当前proc文件对应的name跟需要查询的是否匹配，匹配则获取对应的pid

### 

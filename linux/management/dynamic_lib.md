# dynamic_lib

## 确定应用所需要依赖的so文件

```c
[johnsonz@sfu03 lib]$ ldd /usr/local/bin/ag 
	linux-gate.so.1 =>  (0x00730000)
	libpcre.so.0 => /lib/libpcre.so.0 (0x034bd000)
	liblzma.so.0 => /usr/lib/liblzma.so.0 (0x00d0e000)
	libz.so.1 => /lib/libz.so.1 (0x007ea000)
	libpthread.so.0 => /lib/libpthread.so.0 (0x0098e000)
	libc.so.6 => /lib/libc.so.6 (0x00200000)
	/lib/ld-linux.so.2 (0x005d0000)
```

## Linux寻找动态库的路径

在 Linux 下面，共享库的寻找和加载是由 /lib/ld.so 实现的。 ld.so 在标准路经(/lib, /usr/lib) 中寻找应用程序用到的共享库。
如果是非标准路径，Linux 通用的做法是将非标准路经加入 /etc/ld.so.conf，然后运行 **ldconfig** 生成 /etc/ld.so.cache。 ld.so 加载共享库的时候，会从 ld.so.cache 查找。
传统上，Linux 的先辈 Unix 还有一个环境变量：LD_LIBRARY_PATH 来处理非标准路经的共享库。ld.so 加载共享库的时候，也会查找这个变量所设置的路经。
LD_LIBRARY_PATH=dir：$LD_LIBRARY_PATH
export LD_LIBRARY_PATH  
但是，有不少声音主张要避免使用 LD_LIBRARY_PATH 变量，尤其是作为全局变量。

```c

```
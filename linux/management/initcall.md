# initcall

## initcall source code in Linux2.6.14
```c
#define __define_initcall(level,fn) \
    static initcall_t __initcall_##fn __attribute_used__ \
    __attribute__((__section__(".initcall" level ".init"))) = fn

#define core_initcall(fn)       __define_initcall("1",fn)
#define postcore_initcall(fn)       __define_initcall("2",fn)
#define arch_initcall(fn)       __define_initcall("3",fn)
#define subsys_initcall(fn)     __define_initcall("4",fn)
#define fs_initcall(fn)         __define_initcall("5",fn)
#define device_initcall(fn)     __define_initcall("6",fn)
#define late_initcall(fn)       __define_initcall("7",fn)


#define __initcall(fn) device_initcall(fn)

#define __exitcall(fn) \
    static exitcall_t __exitcall_##fn __exit_call = fn

#define console_initcall(fn) \
    static initcall_t __initcall_##fn \
    __attribute_used__ __attribute__((__section__(".con_initcall.init")))=fn

#define security_initcall(fn) \
    static initcall_t __initcall_##fn \
    __attribute_used__ __attribute__((__section__(".security_initcall.init"))) = fn

	
#define module_init(x)  __initcall(x);
#define module_exit(x)  __exitcall(x);
```
在第一段代码中
```c
#define __define_initcall(level,fn) \
    static initcall_t __initcall_##fn __attribute_used__ \
    __attribute__((__section__(".initcall" level ".init"))) = fn
```
表达了两个关键的信息，
- __initcall_fn = fn
- __initcall_fn 要放在.initcallx.init 段中。

下面是在Linux2.6.14中对应的lds文件：

```c
SECTIONS
{
 . = 0xC0008000;
 .init : { /* Init code and data        */
  _stext = .;
   _sinittext = .;
   *(.init.text)
   _einittext = .;
  __proc_info_begin = .;
   *(.proc.info.init)
  __proc_info_end = .;
  __arch_info_begin = .;
   *(.arch.info.init)
  __arch_info_end = .;
  __tagtable_begin = .;
   *(.taglist.init)
  __tagtable_end = .;
  . = ALIGN(16);
  __setup_start = .;
   *(.init.setup)
  __setup_end = .;
  __early_begin = .;
   *(.early_param.init)
  __early_end = .;
  __initcall_start = .;
   *(.initcall1.init)
   *(.initcall2.init)
   *(.initcall3.init)
   *(.initcall4.init)
   *(.initcall5.init)
   *(.initcall6.init)
   *(.initcall7.init)
  __initcall_end = .;
```


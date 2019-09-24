# U-boot

## U-boot build
Download Website: https://ftp.denx.de/pub/u-boot/
**Note: Usually, we need to build U-boot for Embedded board, arm or mips cpu, so we need to make use of arm-linux-gcc build tool chain**

**Based on U-Boot 1.3.2 Version**

- clean the u-boot: make distclean
- generate kernel config file: make smdk2410_config
- make

### Fix Build error

**board.c:129: error: inline function 'coloured_LED_init' cannot be declared weak**

commnet the related line in board.c

**inline function 'show_boot_progress' cannot be declared weak**

remove the 'inline' keyword

**arm920t/start.S:119: undefined reference to 'coloured_LED_init'**
remove this call

## Linux Build

### Modify Makefile
ARCH            ?= arm
CROSS_COMPILE   ?= arm-linux-

### menuconfig
make s3c2410_config

### make

make

## initrd.img 制作

- mkdir initrd
- dd if=/dev/zero of=initrd.img bs=1k count=4096
- mke2fs -F -v initrd.img
- mount -o loop initrd.img initrd
- cd initrd
- cp -r ../_install/ .
- mkdir proc lib etc dev root home var tmp
- chmod 777 tmp
- cd dev
- mknod -m 644 console c 5 1
- mknod -m 644 null c 1 3
- mknod -m 640 ram b 1 1
- mknod -m 644 mem c 1 1
- cd ..
- create "etc/inittab"
```c
::sysinit:/etc/init.d/rcS
::askfirst:-/bin/sh
::restart:/sbin/init
::ctrlaltdel:/sbin/reboot
::shutdown:/bin/umount -a -r
::shutdown:/sbin/swapoff -a
```
- chmod 644 etc/inittab
- etc/init.d/rcS
```c
#!/bin/sh
/bin/mount -t proc none /proc
/sbin/ifconfig lo 127.0.0.1 up
/sbin/ifconfig eth0 10.0.0.2 netmask 255.0.0.0 up
hostname skyeye
mkdir /var/tmp
mkdir /var/log
mkdir /var/run
mkdir /var/lock
/bin/ash
```
- chmod 755 etc/init.d/rcS
- cd ..
- umount initrd

### dd command

### mke2fs


## UBOOT 源码

### start.S

### stage 2
```c
void start_armboot (void)
{
 /*   TEXT
 *    HEAP  
 *    GLOBAL DATA TABLE
 *  
 *    armboot_start is the base address of TEXT.
 */
    /* Pointer is writable since we allocated a register for it */
    gd = (gd_t*)(_armboot_start - CFG_MALLOC_LEN - sizeof(gd_t));
    /* compiler optimization barrier needed for GCC >= 3.4 */
    __asm__ __volatile__("": : :"memory");
    
    memset ((void*)gd, 0, sizeof (gd_t));
    gd->bd = (bd_t*)((char*)gd - sizeof(bd_t));
    memset (gd->bd, 0, sizeof (bd_t));
            
    monitor_flash_len = _bss_start - _armboot_start;
    
    for (init_fnc_ptr = init_sequence; *init_fnc_ptr; ++init_fnc_ptr) {
        if ((*init_fnc_ptr)() != 0) {
            hang ();
        }
    }

}

init_fnc_t *init_sequence[] = {
    cpu_init,       /* basic cpu dependent setup */
    board_init,     /* basic board dependent setup */
    interrupt_init,     /* set up exceptions */
    env_init,       /* initialize environment */
    init_baudrate,      /* initialze baudrate settings */
    serial_init,        /* serial communications setup */
    console_init_f,     /* stage 1 init of console */
    display_banner,     /* say that we are here */
#if defined(CONFIG_DISPLAY_CPUINFO)
    print_cpuinfo,      /* display cpu info (and speed) */
#endif
#if defined(CONFIG_DISPLAY_BOARDINFO)
    checkboard,     /* display board info */
#endif
#if defined(CONFIG_HARD_I2C) || defined(CONFIG_SOFT_I2C)
    init_func_i2c,
#endif
    dram_init,      /* configure available RAM banks */
    display_dram_config,
    NULL,
};

```


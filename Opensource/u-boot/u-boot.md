# U-boot

## U-boot build
Download Website: https://ftp.denx.de/pub/u-boot/
**Note: Usually, we need to build U-boot for Embedded board, arm or mips cpu, so we need to make use of arm-linux-gcc build tool chain**

**Based on U-Boot 1.3.2 Version**


- generate kernel config file: make smdk2410_config
- make

### Fix Build error

**board.c:129: error: inline function 'coloured_LED_init' cannot be declared weak**

commnet the related line in board.c

**inline function 'show_boot_progress' cannot be declared weak**

remove the 'inline' keyword

**arm920t/start.S:119: undefined reference to 'coloured_LED_init'**
remove this call
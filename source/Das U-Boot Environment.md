
**Das U-Boot** uses a small amount of space on the flash storage usually on the same partition it is stored on to store some important configuration parameters. This can hardly be compared to NVRAM/TFFS-approach of other bootloaders. It is called the u-boot environment. It stores some values like the IP address of the TFTP server (on your PC) to which the the TFTP client (part of U-Boot) will try to connect, etc.

You can read and write these values when you are connected to the U-Boot console via [Serial Port](http://wiki.openwrt.org/doc/hardware/port.serial) and also from the CLI once you booted OpenWrt.

One of the huge advantages of [Das U-Boot](http://wiki.openwrt.org/doc/techref/bootloader/uboot) is its ability for run time configuration. This flexibility is based on being able to easily change environment variables. The environment is usually at the end of the uboot [partition](http://wiki.openwrt.org/doc/techref/flash.layout). The environment variables are set up in a **board specific file**, e.g. [package/boot/uboot-ar71xx/files/include/configs/nbg460n.h](https://dev.openwrt.org/browser/trunk/package/boot/uboot-ar71xx/files/include/configs/nbg460n.h) for the [Zyxel NBG 460N/550N/550NH](http://wiki.openwrt.org/toh/zyxel/nbg460n).

The location on the flash partition is predefined:

```
#define CONFIG_ENV_OFFSET                 0x0000
#define CONFIG_ENV_SIZE                   0x2000
```

and copied to RAM when U-Boot starts.

The U-Boot Environment is protected by a [CRC32](http://en.wikipedia.org/wiki/Cyclic%20redundancy%20check) checksum. 
See [** Warning - bad CRC, using default environment](http://www.denx.de/wiki/view/DULG/WarningBadCRCUsingDefaultEnvironment)


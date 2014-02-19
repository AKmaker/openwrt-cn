
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

### Common variables
<http://www.denx.de/wiki/view/DULG/UBootEnvVariables>

This lists the most important environment variables, all of which have a special meaning to U-Boot.

Variable    | Description
------------|------------------------------------------------------------------------
autoload    | if set to no (or any string beginning with 'n'), the rarpb, bootp or dhcp commands will perform only a configuration lookup from the BOOTP / DHCP server, but not try to load any image using TFTP.
autostart   | if set to yes, an image loaded using the rarpb, bootp, dhcp, tftp, disk, or docb commands will be automatically started (by internally calling the bootm command).
baudrate    | a decimal number that selects the console baudrate (in bps).
bootcmd     | This variable defines a command string that is automatically executed when the initial countdown is not interrupted. This command is only executed when the variable bootdelay is also defined!
bootdelay   | After reset, U-Boot will wait this number of seconds before it executes the contents of the bootcmd variable. During this time a countdown is printed, which can be interrupted by pressing any key. Set this variable to 0 boot without delay. Be careful: depending on the contents of your bootcmd variable, this can prevent you from entering interactive commands again forever! Set this variable to -1 to disable autoboot.
bootfile    | name of the default image to load with TFTP
ethaddr     | Ethernet MAC address for first/only ethernet interface (eth0 in Linux). 
This variable can be set only once (usually during manufacturing of the board). U-Boot refuses to delete or overwrite this variable once it has been set.
ipaddr      | IP address; needed for tftp command
loadaddr    | Default load address for commands like tftp or loads
serverip    | TFTP server IP address; needed for tftp command.
silent      | If the configuration option CONFIG_SILENT_CONSOLE has been enabled for your board, setting this variable to any value will suppress all console messages. Please see [silent_booting](http://wiki.openwrt.org/silent_booting) for details.
verify      | If set to n or no disables the checksum calculation over the complete image in the bootm command to trade speed for safety in the boot process. Note that the header checksum is still verified.

### Accessing U-Boot environment variables in Serial Console

### Accessing U-Boot environment variables in Net Console

### Accessing U-Boot environment variables in OpenWrt
[All changes you make to the U-Boot environment are made in RAM only!](http://www.denx.de/wiki/view/DULG/UBootCmdGroupEnvironment) If you want to additionally make your changes permanent you have to use the saveenv command to write a copy of the environment settings from RAM to persistent storage. The saveenv command is not available in the opkg-package **uboot-envtools**, also, the bootloader partition will likely be mounted read-only. An example on how to change this, is here: [making.bootloader.partition.writable](http://wiki.openwrt.org/toh/tp-link/tl-wr1043nd#making.bootloader.partition.writable)

### Refs

* <http://wiki.openwrt.org/doc/techref/bootloader/uboot.config>


The Bootloader is a piece of software that is executed every time the hardware device is powered up. It is executable machine code and thus ARCH-specific. It's quite heavily device-specific because its main task is to initialize all the low-level hardware details. The bootloader can be contained on a separate EEPROM (very seldom) or directly on flash storage (most common).

NOTE:
Being a piece of software, the bootloader is considered part of the firmware, but **the bootloader is not part of OpenWrt!** Only on seldom occasions a change of the bootloader settings or the bootloader code is necessary to allow for booting/installing OpenWrt.There are a number of bootloaders under diverse software licenses.

### Main Function

The bootloader's main function is to initialize the hardware, pass an abstraction of the initialized hardware, a hardware description, to and execute the Kernel. (A very nice technical example can be seen [here](http://www.wehavemorefun.de/fritzbox/index.php/ADAM2#Funktionen).) After that the bootloader is done and not needed in memory any longer. Most bootloaders offer [additional functions](http://wiki.openwrt.org/doc/techref/bootloader#additional.functions).

#### Why this is necessary?

It's not. A bootloader is not required to boot Linux. The use of one (or several) bootloaders in a row to chainload (or bootstrap) a Kernel is not a categorical necessity, it is merely a very crafty method to start an operating system. The main advantage for OpenWrt is, that the existence of a bootloader offers users and developers additional possibilities to **debrick a device**.

### Features

#### Limitations

Some bootloader or implementation of universal bootloaders come with certain limitation implemented by the OEM, such as:
* a willfully designed kernel/firmware size limitation
* make the bootloader expect a certain exotic firmware format
* inability of the bootloader to execute the ELF binary format
* the need of some obscure "magic value" to be present and correct
* you name it

The reasons are variable, from simple ineptitude of the creators to the willful sabotage of the users' attempts to run free software on their own property. 

#### Additional Functions

The bootloader can be more or less sophisticated, and offer none to many additional functions. In many situations additional functions would give the user a huge advantage, so most bootloaders offer them, such as:

* Flash new firmware, see [generic.flashing](http://wiki.openwrt.org/doc/howto/generic.flashing)
* The bootloader can possibly validate data on the flash-storage, and e.g. should the firmware fail to pass a CRC, the bootloader would presume the firmware is corrupt and wait for a new firmware to be uploaded over the network. Of course, this also means, that every time you change something on the flash, you have to update this CRC-value…
* this could also be triggered by setting a boot_wait variable or by a command executed in the serial console.
* a CLI(aka serial console), which usually can only be accessed over the serial port only. There are several ways to access the serial console port on your target system, such as using a terminal server, but the most common way is to attach it to a [serial port](http://wiki.openwrt.org/doc/hardware/port.serial) on your host. Additionally, you will need a terminal emulation program on your host system, such as **cu** or **kermit**.
* once the bootloader has fulfilled its main function - chainload the Kernel - it does not run any longer, so to play with it, you have to login to it before it loads the Kernel and you may also have to prevent it from loading the Kernel, i.e. to stop the boot process.
* boot from USB:
Like the Kernel requires the module **kmod-fs-ext4** to read/write to a EXT4 filesystem, so a bootloader requires such a module to do the same. GRUB2 has this functionality implemented, so with it, you can very comfortably configure your boot options and also update and maintain your OS. The lightweight bootloaders we use with OpenWrt, usually do not have this functionality. But see [Flash Layout](http://wiki.openwrt.org/doc/techref/flash.layout) for further reference. One exception is the U-Boot implementation of the **dockstar**. It can not only initialize the USB (like all the rest of the hardware) but additionally utilize the USB and also understand the EXT2 filesystem. Thus, the dockstar can be booted directly from an ext2-formated harddisc/usb-stick connected to any of it's USB-ports.
* [net booting](http://wiki.openwrt.org/inbox/netboot) functionality via BOOTP or PXE or DHCP or NFS or TFTP

### Boot Procedure

[boot process](http://wiki.openwrt.org/doc/techref/process.boot) should give a more detailed description of whole boot procedure. The bootloader is the beginning. 

#### Individual Bootloaders

##### PC

An **embedded bootloader** fulfills the same functionality as the **BIOS** plus **GNU GRUB** together on a PC.

* BIOS(proprietary): the BIOS of your PC is nothing else but a bootloader!
* UEFI(proprietary): successor want-to-be of the BIOS
* coreboot(GPLv2): successor of the BIOS, alternative to UEFI based on the Linux kernel;
 support for x86, x86-64 and ARM. There is no MIPS support.
 Coreboot does only "a little bit of hardware initialization"
* GNU GRUB(GPLv2)

##### Embedded Devices

* Das U-Boot(GPLv2): arguably the richest, most flexible, and most actively developed FOSS bootloader available
* RedBoot(mod GPL)
* CFE(BSD like): by Broadcom; the orange color means, the OEM is not obliged to deliver the source code.
* Adam2(proprietary): for AR7/UR8
* PSPBoot(proprietary): the only slightly compatible successor of Adam2.
* brnboot(unknown): sometimes called AMAZON Loader.
* yamon(unknown): by Imagination Technology; the Linux kernel can only be booted when it is in SREC format.
* Bootbase(unknown): used by the ZyXEL Prestige 660HW-xx and Prestige 660M-xx devices (and probably other ZyXEL products). <http://www.ixo.de/info/zyxel_uclinux/>
* MyLoader(unknown)
* PP_Boot(unknown)
* VxWorks' own bootloader: most Atheros devices (There is a description of the basic workings on the Netgear WGT624 page).
* NetBoot: the standard loader in DWL7100AP allows to boot firmware image via network from TFTP server direct to RAM.
* ThreadX: D-Link uses OS called **ThreadX** on lowend 1MiB Flash storage & 8MiB RAM models. They have custom boot loader that doesn't output anything sensible to serial port but does have recovery mode so you can upload firmware using browser.

### Refs

* <http://wiki.openwrt.org/doc/techref/bootloader>

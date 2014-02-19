
### OpenWrt

OpenWrt是一个高度模块化, 高度自动化的嵌入式Linux系统, 拥有强大的网络组件, 常常被用于工控设备, 电话, 小型机器人, 智能家居, 路由器以及VOIP设备中. OpenWrt支持各种处理器架构，无论是对ARM，X86，PowerPC或者MIPS都有很好的支持. 其多达3000多种软件包, 囊括从工具链(toolchain), 到内核(linux kernel), 到软件包(packages), 再到根文件系统(rootfs)整个体系, 使得用户只需简单的一个make命令即可方便快速地定制一个具有特定功能的嵌入式系统来制作固件. 其模块化设计也可以方便的移植各类功能到OpenWrt下, 加快开发速度.

对于开发人员, OpenWrt是使用框架来构建应用程序, 而无需建立一个完整的固件来支持. 对于用户来说, 这意味着其拥有完全定制的能力, 可以用前所未有的方式使用该设备.

![openwrt-logo](openwrt-logo.png)

2014年12月19日小米路由器公测版正式发售, 也意味着OpenWrt进入国内主流科技企业的眼球. 然而OpenWrt到底是一款什么样的操作系统呢? 对于创客来讲, 怎么才能融入创客的设计, 下面就从零介绍如果在pcDuino上开发OpenWrt.

OpenWrt项目始于2004年1月. 最早的OpenWrt版本基于Linksys为遵守GPL而放出的, 为WRT54G所编写的代码, 以及uclibc项目的buildroot. 这个版本以OpenWrt "stable release"之名为人所知, 使用广泛. 仍有许多OpenWrt应用程序是基于这一版的, 例如Freifunk-Firmware和Sip@Home.

2005年初, 一些新的开发者进入了团队. 在封闭开发了数月之后, 团队决定发布OpenWrt的第一个experimental版本. 这个实验版本使用的build系统是基于buildroot2大改而成的, 而buildroot2来自于uclibc项目. OpenWrt使用官方版GNU/Linux内核代码, 只是额外添加了片上系统(SoC)的补丁和网络接口的驱动. 开发团队尝试重新实现GPL tarball中不同开发商的绝大多数专有代码. 其中有: 将新固件镜像文件直接写入闪存的自由工具(mtd), 配置无线局域网(wlcompat/wificonf), 通过proc文件系统对支持VLAN的switch(交换机)进行编程. 最初发布的OpenWrt的代号是"White Russian", 来自于著名鸡尾酒的名称. 在OpenWrt发布0.9版的时候, White Russian的生命周期结束.

下一个版本的开发正在我们的SVN中进行. 下面一张图将很清晰的反映OpenWrt的版本史. 从图上可以看出最新的稳定版本代号为Attitude Adjustment, 早期的稳定版本为Backfire 和 Kamikaze, 开发版本一直都是trunk. 各个版本的官方下载地址为 <https://dev.openwrt.org/wiki/GetSource>

![code_overview](code_overview.png)

### 下载编译OpenWrt

1. 安装依赖包(deb系列)

```bash
sudo aptitude install -y libncurses5-dev zlib1g-dev gawk flex patch git-core g++ subversion
```

2. OpenWrt的源代码管理默认用的是SVN, 当然你还可以用Git, 本教程中使用最新的trunk版本, 用SVN工具下载源码

```bash
svn co svn://svn.openwrt.org/openwrt/trunk/ openwrt-pcduino
```

还可以用Git下载

```bash
git clone git://git.openwrt.org/openwrt.git
git clone git://git.openwrt.org/packages.git
```

3. 扩展软件包package feeds, feeds即为包含到你的OpenWrt环境中的额外软件包的软件列表索引(类似linux发行版中的软件源, 不过此处为软件包的源码). 目前常用的feeds有:

```bash
src-git packages git://git.openwrt.org/packages.git
src-svn xwrt http://x-wrt.googlecode.com/svn/trunk/package
src-git luci git://nbd.name/luci.git
src-git routing git://github.com/openwrt-routing/packages.git
src-git telephony http://feeds.openwrt.nanl.de/openwrt/telephony.git
src-svn phone svn://svn.openwrt.org/openwrt/feeds/phone
src-svn efl svn://svn.openwrt.org/openwrt/feeds/efl
src-svn xorg svn://svn.openwrt.org/openwrt/feeds/xorg
src-svn desktop svn://svn.openwrt.org/openwrt/feeds/desktop
src-svn xfce svn://svn.openwrt.org/openwrt/feeds/xfce
src-svn lxde svn://svn.openwrt.org/openwrt/feeds/lxde
src-link custom /usr/src/openwrt/custom-feed
```

一般情况, 你至少需要含packages feeds, 其他可根据需求下载, 安装feeds.
* packages – 提供众多库, 工具等基本功能. 也是其他feed所依赖的软件源, 因此在安装其他feed前一定要先安装packages!
* luci – OpenWrt默认的GUI(WEB管理界面).
* xwrt – 另一种可替换LuCI的GUI
* qpe – DreamBox维护的基于Qt的图形界面, 包含Qt2, Qt4, Qtopia, OPIE, SMPlayer等众多图形界面.
* device – DreamBox维护与硬件密切相关的软件, 如uboot, qemu等.
* dreambox_packages – DreamBox维护的国内常用网络工具, 如oh3c, njit8021xclient等.
* desktop - OpenWrt用于桌面的一些软件包.
* xfce - 基于Xorg的著名轻量级桌面环境. Xfce建基在GTK+2.x之上, 它使用Xfwm作为窗口管理器.
* efl - 针对enlightenment.
* phone -针对fso, paroli.

Trunk中默认的feeds下载有packages、xwrt、luci、routing、telephony。如果你需要其他的软件包，你只需要打开源码根目录下面的feeds.conf.default文去掉你需要的软件包前面的#号，本教程中使用默认的软件，确定了软件源之后，更新源:

```bash
./scripts/feeds update -a
```

安装下载好的包:

```bash
./scripts/feeds install -a
```

4. OpenWrt源码目录结构, 执行上面命令之后你就可以得到全部的Openwrt源码.

目录结构:

* tools和toolchain包含了一些通用命令, 用来生成固件, 编译器, 和C库.
* build dir/host是一个临时目录, 用来储存不依赖于目标平台的工具.
* build dir/toolchain-<arch>用来储存依赖于指定平台的编译链. 只是编译文件存放目录无需修改.
* build dir/target-<arch>用来储存依赖于指定平台的软件包的编译文件, 其中包括linux内核, u-boot, packages, 只是编译文件存放目录无需修改.
* staging_dir是编译目标的最终安装位置, 其中包括rootfs, package, toolchain.
* package软件包的下载编译规则, 在OpenWrt固件中, 几乎所有东西都是.ipk, 这样就可以很方便的安装和卸载.
* target目标系统指嵌入式设备, 针对不同的平台有不同的特性, 针对这些特性, "target/linux"目录下按照平台<arch>进行目录划分, 里面包括了针对标准内核的补丁, 特殊配置等.
* bin编译完OpenWrt的二进制文件生成目录, 其中包括sdk, uImage, u-boot, dts, rootfs构建一个嵌入式系统完整的二进制文件.
* config存放着整个系统的的配置文件.
* docs里面不断包含了整个宿主机的文件源码的介绍, 里面还有Makefile为目标系统生成docs.
* include里面包括了整个系统的编译需要的头文件, 但是是以Make进行连接的.
* feeds扩展软件包索引目录.
* scripts组织编译整个OpenWrt的规则.
* tmp编译文件夹, 一般情况为空.
* dl所有软件的下载目录, 包括u-boot, kernel.
* logs如果编译出错, 可以在这里找到编译出错的log.

5. 配置OpenWrt编译系统

```bash
make menuconfig
```

![openwrt-buildroot](openwrt-buildroot.png)

在官方最新的trunk分支中已经支持pcDuino这个target了.

具体配置如下:
A. 配置目标系统(Target System)

```
Target System (Allwinner A1x/A20/A3x) —> 
```

B. 配置目标硬件(Target Profile) 

```
Target Profile (pcDuino) —> 
```

C. 配置编译出来的image, 配置rootfs文件系统的格式这里选择ext4, rootfs文件系统的大小这里设置(48M).

```
Target Images  —> 
[ ] ramdisk  —> 
*** Root filesystem archives *** 
[ ] cpio.gz
[*] tar.gz 
*** Root filesystem images *** 
[*] ext4
[ ] jffs2
[ ] squashfs
[*] GZip images
*** Image Options ***
(48) Root filesystem partition size (in MB)
(6000) Maximum number of inodes in root filesystem
(0) Percentage of reserved blocks in root filesystem
[ ] Include kernel in root filesystem  —>
[ ] Include DTB in root filesystem
```

D. 选择编译交叉编译器, 还有开发SDK.

```
[*] Build the OpenWrt Image Builder
[*] Build the OpenWrt SDK
```

E. 配置无线网卡, V2/V3都是用的rtl8188cus无线网卡

```
Kernel modules  —>
Wireless Drivers  —>
-*- kmod-cfg80211…………………. cfg80211 – wireless configuration API
<*> kmod-lib80211……………………………… 802.11 Networking stack
{M} kmod-mac80211………………… Linux 802.11 Wireless Networking Stack
<M> kmod-rtl8192cu………………….. Realtek RTL8192CU/RTL8188CU support
{M} kmod-rtlwifi……………………………. Realtek common driver part 
```

F. LucI系统快速配置接口 

```
LuCI  —>

1. Collections  —>
{*} luci
<M> luci-ssl……………………. Standard OpenWrt set with HTTPS

4. Themes  —> 
-*- luci-theme-base…………………………. Common base for all
-*- luci-theme-bootstrap……………………… Bootstrap Theme
<*> luci-theme-freifunk-bno……………….. Freifunk Berlin Nordost Theme
<*> luci-theme-freifunk-generic………………….. Freifunk Generic Theme
<*> luci-theme-openwrt……………………………………. OpenWrt.org

5. Translations  —>
<*> luci-i18n-chinese………………….. Chinese (by Chinese Translators)
-*- luci-i18n-english………………………………………… English
```

6. 编译OpenWrt系统

```bash
make –j 8 V=s 
```

由于OpenWrt整个系统非常庞大, 编译很慢. "-j 8" 表示用8线程进行编译, "V=s"编译的时候显示编译信息. 如果你的电脑是4核建议你用8线程进行编译, 双核建议你使用4线程. 这里测试8线程编译需要一个小时才能编译完成. 

### 构建pcDuino BSP 

由于OpenWrt的u-boot用的是u-boot-2013的版本, 目前只支持SD卡启动, 而且内核用的是3.12.5版本. 另外我们的3.4.29的内核用的是全志fex, 而且3.12.5用的是linux官方的kernel使用的是dts设备树.这样的话我们就不能用之前的BSP方案, 我们要自己做一个从SD卡启动的系统.

pcDuino从SD卡启动顺序是A10—>u-boot–>uImage–>OpenWrt.

根据全志官网的说明, 这些软件都必须放在SD卡固定的地址. 那么首先要对A10进行分区. 根据全志芯片的说明, 需要对SD卡进行下表固定分区.

![a10-flash](a10-flash.png)

1. 先格式化TF卡前面的1M空间, 这里是将TF卡通过读卡器插入到PC的虚拟机. 可以看出TF卡的设备是sdb.

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       195G   60G  126G  33% /
udev            989M  4.0K  989M   1% /dev
tmpfs           400M  940K  399M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            998M   76K  998M   1% /run/shm
/dev/sdb1       3.8G   12K  3.8G   1% /media/0005-559B

sudo dd if=/dev/zero of=/dev/sdb bs=1M count=1
```

2. 写入u-boot-spl.bin和u-boot.bin. OpenWrt的生成的二进制文件都在openwrt/trunk/bin/sunxi目录下, 这里OpenWrt做了一些工作将u-boot-spl.bin和u-boot.bin合在了一起, 只需要把openwrt-sunxi-pcDuino-sunxi-with-spl.bin写到 

```
cd  uboot-sunxi-pcDuino
pillar@monster:~/openwrt/trunk/bin/sunxi/uboot-sunxi-pcDuino$ ls 
openwrt-sunxi-pcDuino-sunxi-spl.bin       openwrt-sunxi-pcDuino-u-boot.bin
openwrt-sunxi-pcDuino-sunxi-with-spl.bin
sudo dd if=openwrt-sunxi-pcDuino-sunxi-with-spl.bin of=/dev/sdb bs=1024 seek=8
```

这时候把SD卡插到板子上, 重新上电就会看到下面打印信息.

```
U-Boot 2013.10-rc2 (Jan 15 2014 – 17:48:38) Allwinner Technology
CPU:   Allwinner A10 (SUN4I)
Board: pcDuino
I2C:   ready
DRAM:  1 GiB
MMC:   SUNXI SD/MMC: 0

*** Warning – bad CRC, using default environment
In:    serial
Out:   serial
Err:   serial
Net:   emac
Hit any key to stop autoboot:  0

sun4i#
```

从上面可以看到u-boot已经完全启动了, 上面的时间是编译的时间. 上面的信息还可以看到我们的环境变量没有设置, 它使用的是默认的环境变量. 前面介绍, 系统建立在分区表不同的地方, 但是我们现在SD卡还没有分区表, 我们需要先建立分区表再做环境变量.

3. 建立分区表. 重新把SD卡插回到电脑的虚拟机里面, 使用fdisk创建分区表. 具体的分区见下操作, 步骤的说明请看#后面的注释.

```bash
pillar@monster:~/openwrt/trunk/bin/sunxi$ sudo fdisk /dev/sdb 
[sudo] password for pillar: 
Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
Building a new DOS disklabel with disk identifier 0x97bf3019.
Changes will remain in memory only, until you decide to write them.
After that, of course, the previous content won’t be recoverable.
Warning: invalid flag 0×0000 of partition table 4 will be corrected by w(rite) 
Command (m for help): m   #帮助 
Command action 
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   l   list known partition types
   m   print this menu
   n   add a new partition   #创建分区
   o   create a new empty DOS partition table
   p   print the partition table  #查看分区
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition’s system id   #改变分区类型
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only) 

Command (m for help): p             #查看分区
Disk /dev/sdb: 4027 MB, 4027580416 bytes
124 heads, 62 sectors/track, 1023 cylinders, total 7866368 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x97bf3019

Device Boot      Start         End      Blocks   Id  System

#没有分区 

Command (m for help): n      #创建分区 

Partition type: 
   p   primary (0 primary, 0 extended, 4 free)    #主分区 
   e   extended                                   #扩展分区 

Select (default p):                            #选择默认主分区
Using default response p
Partition number (1-4, default 1):                #分区号为1
Using default value 1
First sector (2048-7866367, default 2048):         #选择默认值
Using default value 2048
Last sector, +sectors or +size{K,M,G} (2048-7866367, default 7866367): 34815 #这个根据全志的手册来第一个分区必须这么大

Command (m for help): p                       #查看分区 

Disk /dev/sdb: 4027 MB, 4027580416 bytes
124 heads, 62 sectors/track, 1023 cylinders, total 7866368 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x97bf3019

   Device Boot      Start         End      Blocks   Id  System 
/dev/sdb1            2048       34815       16384   83  Linux  #创建的第一个分区

Command (m for help): n                    #再创建一个分区 
Partition type:
   p   primary (1 primary, 0 extended, 3 free) 
   e   extended 

Select (default p):                          #主分区
Using default response p 
Partition number (1-4, default 2):              #第二个主分区
Using default value 2
First sector (34816-7866367, default 34816):      #默认大小从34816开始
Using default value 34816
Last sector, +sectors or +size{K,M,G} (34816-7866367, default 7866367): #默认全部分到第二分区 

Using default value 7866367 

Command (m for help): p                       #再一次查看分区 

Disk /dev/sdb: 4027 MB, 4027580416 bytes
124 heads, 62 sectors/track, 1023 cylinders, total 7866368 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x97bf3019

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1            2048       34815       16384   83  Linux
/dev/sdb2           34816     7866367     3915776   83  Linux

#可以看出创建了两个分区都为linux类型，但是u-boot只能识别第一个分区为FAT32分区 

Command (m for help): t            #修改分区类型
Partition number (1-4): 1            #选择修改哪个分区
Hex code (type L to list codes): L      #列出所有类型

 0  Empty           24  NEC DOS         81  Minix / old Lin bf  Solaris
 1  FAT12           27  Hidden NTFS Win 82  Linux swap / So c1  DRDOS/sec (FAT-
 2  XENIX root      39  Plan 9          83  Linux           c4  DRDOS/sec (FAT-
 3  XENIX usr       3c  PartitionMagic  84  OS/2 hidden C:  c6  DRDOS/sec (FAT-
 4  FAT16 <32M      40  Venix 80286     85  Linux extended  c7  Syrinx
 5  Extended        41  PPC PReP Boot   86  NTFS volume set da  Non-FS data
 6  FAT16           42  SFS             87  NTFS volume set db  CP/M / CTOS / .
 7  HPFS/NTFS/exFAT 4d  QNX4.x          88  Linux plaintext de  Dell Utility
 8  AIX             4e  QNX4.x 2nd part 8e  Linux LVM       df  BootIt
 9  AIX bootable    4f  QNX4.x 3rd part 93  Amoeba          e1  DOS access
 a  OS/2 Boot Manag 50  OnTrack DM      94  Amoeba BBT      e3  DOS R/O
 b  W95 FAT32       51  OnTrack DM6 Aux 9f  BSD/OS          e4  SpeedStor
 c  W95 FAT32 (LBA) 52  CP/M            a0  IBM Thinkpad hi eb  BeOS fs
 e  W95 FAT16 (LBA) 53  OnTrack DM6 Aux a5  FreeBSD         ee  GPT
 f  W95 Ext’d (LBA) 54  OnTrackDM6      a6  OpenBSD         ef  EFI (FAT-12/16/
10  OPUS            55  EZ-Drive        a7  NeXTSTEP        f0  Linux/PA-RISC b
11  Hidden FAT12    56  Golden Bow      a8  Darwin UFS      f1  SpeedStor
12  Compaq diagnost 5c  Priam Edisk     a9  NetBSD          f4  SpeedStor
14  Hidden FAT16 <3 61  SpeedStor       ab  Darwin boot     f2  DOS secondary
16  Hidden FAT16    63  GNU HURD or Sys af  HFS / HFS+      fb  VMware VMFS
17  Hidden HPFS/NTF 64  Novell Netware  b7  BSDI fs         fc  VMware VMKCORE
18  AST SmartSleep  65  Novell Netware  b8  BSDI swap       fd  Linux raid auto
1b  Hidden W95 FAT3 70  DiskSecure Mult bb  Boot Wizard hid fe  LANstep
1c  Hidden W95 FAT3 75  PC/IX           be  Solaris boot    ff  BBT
1e  Hidden W95 FAT1 80  Old Minix

Hex code (type L to list codes): c             #选择FAT32
Changed system type of partition 1 to c (W95 FAT32 (LBA))

Command (m for help): p                  #再一次查看分区 

Disk /dev/sdb: 4027 MB, 4027580416 bytes
124 heads, 62 sectors/track, 1023 cylinders, total 7866368 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk identifier: 0x97bf3019

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1        2048       34815     16384    c  W95 FAT32 (LBA)#已经修改过了了
/dev/sdb2           34816     7866367     3915776   83  Linux

Command (m for help): w                                   #保存分区表
The partition table has been altered! 
Calling ioctl() to re-read partition table.
WARNING: If you have created or modified any DOS 6.x
partitions, please see the fdisk manual page for additional
information.
Syncing disks.
```

4. 格式化分区. 刚才创建了分区, 但是没有格式化, 我们还是不能使用.

```
pillar@monster :~/openwrt/trunk/bin/sunxi$ ls /dev/sdb  #查看已经分好的分区
sdb   sdb1  sdb2
pillar@monster :~/openwrt/trunk/bin/sunxi$ mkf        #查看有哪些分区类型
mkfifo        mkfontscale   mkfs.bfs      mkfs.ext2     mkfs.ext4     mkfs.minix    mkfs.ntfs
mkfontdir     mkfs          mkfs.cramfs   mkfs.ext3     mkfs.ext4dev  mkfs.msdos    mkfs.vfat 

pillar@monster :~/openwrt/trunk/bin/sunxi$ sudo mkfs.vfat /dev/sdb1 #第一个分区格式化为fat分区
[sudo] password for pillar:
mkfs.vfat 3.0.12 (29 Oct 2011)

pillar@monster :~/openwrt/trunk/bin/sunxi$ sudo mkfs.ext4 /dev/sdb2 #第二个分区格式化为ext4分区，这里需要几分钟

mke2fs 1.42 (29-Nov-2011)
Filesystem label=
OS type: Linux
Block size=4096 (log=2)
Fragment size=4096 (log=2)
Stride=0 blocks, Stripe width=0 blocks
244800 inodes, 978944 blocks
48947 blocks (5.00%) reserved for the super user
First data block=0
Maximum filesystem blocks=1002438656
30 block groups
32768 blocks per group, 32768 fragments per group
8160 inodes per group
Superblock backups stored on blocks:
         32768, 98304, 163840, 229376, 294912, 819200, 884736

Allocating group tables: done
Writing inode tables: done
Creating journal (16384 blocks): done
Writing superblocks and filesystem accounting information: done
```

5. 挂载分区

```
pillar@monster :~/openwrt/trunk/bin/sunxi$ sudo mount /dev/sdb1 /media/1
pillar@monster :~/openwrt/trunk/bin/sunxi$ sudo mount /dev/sdb2 /media/2
pillar@monster :~/openwrt/trunk/bin/sunxi$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1       195G   60G  126G  33% /
udev            989M  4.0K  989M   1% /dev
tmpfs           400M  944K  399M   1% /run
none            5.0M     0  5.0M   0% /run/lock
none            998M   76K  998M   1% /run/shm
/dev/sdb1        16M     0   16M   0% /media/1
/dev/sdb2       3.7G  7.5M  3.5G   1% /media/2
```

6. 制作u-boot环境变量文件. 刚刚创建了分区, 这里只需要将环境变量文件, 还有uImage拷贝到第一分区让u-boot读取, 就可以引导系统了. 下面开始制作u-boot环境变量文件. 

```
pillar@monster :/media/1$ vim boot.cmd
  1 setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10 ${extra}
  2 fatload mmc 0 0×46000000 uImage
  3 fatload mmc 0 0×49000000 sun4i-a10-pcduino.dtb
  4 fdt_high ffffffff
  5 bootm 0×46000000 – 0×49000000
pillar@monster :/media/1$mkimage -C none -A arm -T script -d boot.cmd boot.scr
```

7. 将系统文件拷贝到第一和第二分区.

```
pillar@monster :/media/1$ cp ~/openwrt/trunk/bin/sunxi/sun4i-a10-pcduino.dtb .
pillar@monster :/media/1$ cp ~/openwrt/trunk/bin/sunxi/openwrt-sunxi-uImage uImage 
pillar@monster :/media/1$ ls           #第一分区文件
boot.scr  sun4i-a10-pcduino.dtb    uImage
pillar@monster :/media$ sudo dd if=~/openwrt/trunk/bin/sunxi/openwrt-sunxi-root.ext4 of=/dev/sdb2 bs=1M                   #拷贝第二分区文件
```
好了, 现在整个的从SD启动的BSP已经最好了.

8. 发布并烧写系统. 现在把系统做好了, 你可以发布你制作的系统, 然后别人可以通过win32diskimager来把你的系统写入到他的SD卡, 他就可以和你一起玩OpenWrt了.

```
pillar@monster :/media$ sudo dd if=/dev/sdc of=OpenWrt.img  bs=4M 
```
现在把OpenWrt.img拷贝到windows上, 把你新的SD卡插到电脑开始用win32diskimager写入 

![win32-disk-imager](win32-disk-imager.png)

### 配置OpenWrt系统

1. 让系统上网

```
vim /etc/config/network
config interface ‘net’
        option ifname ‘eth0′
        option proto ‘dhcp’
```

2. 设置固定的mac地址

当系统的启动的时候发现mac地址老是在变，这就会出现一个问题, 有时候能获取到ip, 有时候获取不到ip. 这里可以做一个系统服务, 让系统开机保存mac地址, 然后再开机的时候恢复之前的mac地址.

1) 在/etc/init.d/mac里面编写如下脚本

```
#!/bin/sh /etc/rc.common
START=18
STOP=91
start() {
if [ -f /mac ]; then
dd if=/mac bs=1 count=17 of=/tmp/mac >/dev/null 2>&1
mac_addr=`cat /tmp/mac`
else
mac_file=/sys/class/net/eth0/address
dd if=$mac_file bs=1 of=/mac count=17 >/dev/null 2>&1
mac_addr=`cat /tmp/mac`
fi
ifconfig eth0 down
ifconfig eth0 hw ether $mac_addr
#if failed, save current mac address
if [ $? -ne 0 ]; then
mac_file=/sys/class/net/eth0/address
dd if=$mac_file bs=1 of=/mac count=17 >/dev/null 2>&1
fi
}
```

2) 指定运行的模式

/etc/rc.d/rc则根据其参数指定的运行模式(运行级别, 你在inittab文件中可以设置)来执行相应目录下的脚本. 凡是以Kxx开头的, 都以stop为参数来调用. 凡是以Sxx开头的, 都以start为参数来调用. 调用的顺序按xx 从小到大来执行. 例如, 假设缺省的运行模式是3, /etc/rc.d/rc就会按上述方式调用.
由于设定mac地址要在network之前. 所以要创建链接:

```
ln  -s   /etc/init.d/mac   /etc/rc.d/S18mac
```

3. 开启wifi

openwrt启动之后输入:

```
root@OpenWrt :/# ifconfig
eth0      Link encap:Ethernet  HWaddr AE:DB:9A:D9:31:DE
inet addr:192.168.1.119  Bcast:192.168.1.255  Mask:255.255.255.0
inet6 addr: fe80::acdb:9aff:fed9:31de/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:696 errors:0 dropped:0 overruns:0 frame:0
TX packets:640 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:86028 (84.0 KiB)  TX bytes:377264 (368.4 KiB)
Interrupt:17 Base address:0×4000 

lo        Link encap:Local Loopback
inet addr:127.0.0.1  Mask:255.0.0.0
inet6 addr: ::1/128 Scope:Host
UP LOOPBACK RUNNING  MTU:65536  Metric:1
RX packets:16 errors:0 dropped:0 overruns:0 frame:0
TX packets:16 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:0
RX bytes:1786 (1.7 KiB)  TX bytes:1786 (1.7 KiB)

wlan0     Link encap:Ethernet  HWaddr 00:7A:03:00:29:F4
inet6 addr: fe80::27a:3ff:fe00:29f4/64 Scope:Link
UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
RX packets:0 errors:0 dropped:0 overruns:0 frame:0
TX packets:7 errors:0 dropped:0 overruns:0 carrier:0
collisions:0 txqueuelen:1000
RX bytes:0 (0.0 B)  TX bytes:864 (864.0 B)
```

确保ethX和wlanX都有. openwrt的root密码是没有设置, 你需要从serial debug进入系统设置root密码, 设置方法如下:

```
passwd  root
```

然后在同一个局域网内你的PC的浏览器上输入: ethX的ip, 这里是192.168.1.119.就会出现下面界面: 

![luci-login](luci-login.png)

输入你刚才设置的密码, 进入系统管理界面, 默认是进入状态标签, 这里你可以看到整个系统的运行的状态.

![system-status](system-status.png)

如果你对当前页面不太习惯, 而且在使用上语言上也有些困难, 你可以进入system标签, 在System Properties里面设置language and style如下图所示. 设置完之后save & apply, 重新刷新一下浏览器就可以使用你设置的语言和主题.

![luci-theme](luci-theme.png)

下面进入网络标签栏设置wifi节点, 这个部分是openwrt比较复杂的一个部分, 这个部分的设置直接决定着你的openwrt能不能使用.

![luci-wifi](luci-wifi.png)

添加新接口, 选择静态ip, 新接口的名称, 你需要用英文自定义一个名字, 在包括一下接口里面选择无线网络. 设置完之后提交, 进入下一个页面继续设置.

![luci-interface](luci-interface.png)

基本设置设置完成之后, 进入防火墙设置, 这里wifi必须选择为lan口. 设置完成之后保存应用. 这个时候你电脑就可以连接使用openwrt这个路由器了.

![pc-wifi](pc-wifi.png)

![luci-wifi-net](luci-wifi-net.png)

点击修改后进入防火墙设置标签栏, 分配防火墙区域为wan, 设置完成之后保存&应用.

![luci-firewall](luci-firewall.png)

### 制作内核补丁

1. 清空恢复上一个全新的内核

```
make target/linux/{clean,prepare} V=s QUILT=1
```

2. 到内核源码目录

```
cd build_dir/target-*/linux-*/linux-3.*
```

3. 建立git代码仓库

```
git init
git add * -f
git commit -am "initial commit"
```

4. 修改你的代码, 这里我给我的代码添加`rtl8188cus`驱动.

```
mkdir drivers/net/wireless/rtl8192cus

cp  /home/pillar/openwrt/openwrt-pcDuino/RTL8188C_8192C_USB_linux_v4.0.2_9000.20130911/driver/rtl8188C_8192C_usb_linux_v4.0.2_9000.20130911/*   drivers/net/wireless/rtl8192cus/ -rf

vim  drivers/net/wireless/Kconfig
284  source “drivers/net/wireless/rtl8192cus/Kconfig”

vim drivers/net/wireless/rtl8192cus/Kconfig
  1 config RTL8192CU_SW
  2 tristate “Realtek 8192C USB WiFi for SW”
  3 depends on USB
  4 select WIRELESS_EXT
  5 select WEXT_PRIV

vim  drivers/net/wireless/Makefile
28             obj-$(CONFIG_RTL8192CU_SW)  += rtl8192cus/

vim  drivers/net/wireless/rtl8192cus/Makefile
575  obj-$(CONFIG_RTL8192CU_SW) := $(MODULE_NAME).o 

579  export CONFIG_RTL8192CU_SW = m
```

5. 建立git分支, 并制作补丁

```
git branch rtl8192
git checkout rtl8192
git add * -f
git commit -a -m  "add rtl8192cus for pcDuino"
git format-patch -M master  #会生成0002-add-rtl8192cus-for-pcDuino.patch 

cp  0001-add-rtl8192cus-for-pcDuino.patch patches/
cd ../../../../
make target/linux/update package/index V=s
cp  build_dir/target-arm_cortex-a8+vfpv3_uClibc-0.9.33.2_eabi/linux-sunxi/linux-3.12.5/patches/0001-add-rtl8192cus-for-pcDuino.patch target/linux/sunxi/patches-3.12/
```

5. 检测是否生效, 执行完之后就是`rtl8192`分支的代码了.

```
make target/linux/{clean,prepare} V=s QUILT=1
```

6. 配置内核应用选项

```
make kernel_menuconfig
         Device Drivers  —> 
                   [*] Network device support  —> 
                                     [*]   Wireless LAN  —> 
                                                          <*>   Realtek 8192C USB WiFi
```

其他的patch的制作方法请参考: <http://wiki.openwrt.org/doc/devel/patches>

### 建立App服务器

OpenWrt通过opkg来管理安装整个系统的软件. 目前有很多OpenWrt的软件源, 但是哪些都是针对于MIPS平台的, pcDuino使用的ARM平台, 我们必须自己搭建软件源. 查看了一下MIPS平台的服务器, 其实很简单的, 就是一个apache服务器, 而且OpenWrt编译完成之后, 在openwrt/trunk/bin/sunxi/packages下面已经生成了软件源. 我们只需要将他们联系起来就行了, 这里是在我的PC的虚拟机上搭建的.

```
# sudo apt-get install apache2
```

修改https的根目录

```
pillar@monster :~/openwrt$ vim /etc/apache2/sites-available/default
4     DocumentRoot /home/pillar/openwrt/trunk/bin/sunxi/
```

重启服务器使修改过的配置生效

```
pillar@monster :~/openwrt$ sudo /etc/init.d/apache2 restart
```

修改pcDuino上OpenWrt的源配置

```
root@OpenWrt :/# vim /etc/opkg.conf
src/gz barrier_breaker http://192.168.1.125/packages
dest root /
dest ram /tmp
lists_dir ext /var/opkg-lists
option overlay_root /overlay
```

上面的IP为我们电脑虚拟机的IP, 下面更新一下软件源.

```
# opkg update
```

### 应用程序开发

OpenWrt上面应用程序开发有两种方式, 一种是利用OpenWrt SDK, 一种是利用OpenWrt源码. 这里主要介绍利用OpenWrt源码, 进行开发应用程序, 制作成ipk软件可以安装.

1. 进入package目录, 创建软件目录

```
#cd   /home/pillar/openwrt/trunk/package
#mkdir example1
```

2. 进入example1目录, 创建Makefile文件和代码路径

```
#cd example1
#touch Makefile
#mkdir  src
```

该Makefile具体内容如下:

```Makefile
#User mode tool example
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
PKG_NAME:=example1
PKG_RELEASE:=1
PKG_BUILD_DIR := $(KERNEL_BUILD_DIR)/$(PKG_NAME)
include $(INCLUDE_DIR)/package.mk

define Package/example1
　SECTION:=utils
　CATEGORY:=Base system
　TITLE:=Build for example1 commands
endef

define Package/example1/description
　This package contains an utility useful to use example1 commands.
endef

define Build/Prepare
　　mkdir -p $(PKG_BUILD_DIR)
　　$(CP) ./src/* $(PKG_BUILD_DIR)/
endef 

target=$(firstword $(subst -, ,$(BOARD)))
MAKE_FLAGS += TARGET="$(target)"
TARGET_CFLAGS += -Dtarget_$(target)=1 -Wall

define Build/example1/compile
　　$(MAKE) -C “$(LINUX_DIR)” \
　　　CROSS_COMPILE=”$(TARGET_CROSS)” \
　　　ARCH=”$(LINUX_KARCH)” \
　　　SUBDIRS=”$(PKG_BUILD_DIR)” \
　　　EXTRA_CFLAGS=”$(BUILDFLAGS)”
endef 

define Package/example1/install
　　$(INSTALL_DIR) $(1)/sbin
　　$(INSTALL_BIN) $(PKG_BUILD_DIR)/example1 $(1)/sbin/
endef 

$(eval $(call BuildPackage,example1))
```

3. 进入src目录, 创建相关源文件 

```
cd src 
touch example１.c Makefile 
```

example１.c　具体内容如下:
```C
#include <stdio.h>
int main(void)
{
　　printf(“Hello, world\n”);
　　return 0;
}
```

Makefile文件具体内容如下:

```Makefile
.NOTPARALLEL: 
#OCTEON_ROOT=$(PWD)/src/ 
CC=~/openwrt/main/staging_dir/toolchain-mips64_gcc-4.4.1_eglibc-2.10.1/usr/bin/mips64-openwrt-linux-gnu-gcc
CFLAGS=-mips64r2 -mabi=64 -march=octeon -mtune=octeon
LFLAGS=
.PHONY: all
all: example1
example1:example1.c
　　${CC} ${CFLAGS} ${LFLAGS} -W -g -Wall -Wno-unused-parameter -DUSE_RUNTIME_MODEL_CHECKS=1 \
　　　　-o $@ example1.c
```

4. 回到主路径/home/pillar/openwrt/trunk/, 编译选项配置保存并编译.

```
make menuconfig
　　Base system —>
　　　example1
```

选项设置为M, 保存退出.
然后编译该模块:

```
make package/example1/compile
```

5. 更新package

```
make package/ example1/install
make package/index
```

### 内核驱动开发
OpenWrt开发内核驱动有多种方式, 前面讲到的制作内核补丁也是一种开发方法. 这里介绍直接在OpenWrt系统上开发内核驱动, 把内核驱动做成ipk软件包的形式.

1. 建立工作目录

```
cd  openwrt/trunk/package
mkdir example
```

2. 进入example目录, 创建Makefile文件和代码路径

```bash
cd example 
mkdir src
vim Makefile
```

```Makefile
# Kernel module example
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/kernel.mk
PKG_NAME:=example
PKG_RELEASE:=1
include $(INCLUDE_DIR)/package.mk

define KernelPackage/example
　　SUBMENU:=Other modules
　　DEPENDS:=@TARGET_octeon
　　TITLE:=Support Module for example
　　AUTOLOAD:=$(call AutoLoad,81,example)
　　FILES:=$(PKG_BUILD_DIR)/example/example.$(LINUX_KMOD_SUFFIX)
endef

define Build/Prepare
　　mkdir -p $(PKG_BUILD_DIR)
　　$(CP) -R ./src/* $(PKG_BUILD_DIR)/
endef 

define Build/Compile
　　$(MAKE) -C “$(LINUX_DIR)” \
　　　　CROSS_COMPILE=”$(TARGET_CROSS)” \
　　　　ARCH=”$(LINUX_KARCH)” \
　　　　SUBDIRS=”$(PKG_BUILD_DIR)/example” \
　　　　EXTRA_CFLAGS=”-g $(BUILDFLAGS)” \
　　　　modules
endef 

$(eval $(call KernelPackage,example))
```

3. 进入src目录, 创建代码路径和相关源文件

```bash
cd src
mkdir example
cd example
vim example.c
```

```C
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

/* hello_init —- 初始化函数, 当模块装载时被调用, 如果成功装载返回0, 否则返回非0值 */ 

static int __init hello_init(void)
{
　　　printk("I bear a charmed life.\n");
　　　return 0;
} 

/ * hello_exit —- 退出函数, 当模块卸载时被调用 */
static void __exit hello_exit(void) 
{
　　　printk("Out, out, brief candle\n");
} 

module_init(hello_init);
module_exit(hello_exit);
MODULE_LICENSE("GPL");
MODULE_AUTHOR("Pillar_zuo");
```

```
vim Kconfig

config EXAMPLE
　　tristate "Just a example"
　　default n
　　help
　　　This is a example, for debugging kernel model.
　　　If unsure, say N.

vim Makefile

obj-m := example.o
```

4. 回到OpenWrt源码根目录下

```
make menuconfig
　　Kernel modules —>
　　　　Other modules —>
　　　　　　kmod-example
```

选项设置为M, 保存退出
然后编译该模块:

```
make package/example/compile
make package/index
```

5. 在OpenWrt系统里面就可以用opkg下载使用了.

### 使用OpenWrt SDK

OpenWrt为了避免每次都重新编译系统, 引入了SDK机制. 我们在发布系统的时候也需要发布SDK, 具体的使用方法请下面例子.

1. 解压SDK

```
pillar@monster :~/openwrt/trunk/bin/sunxi$ tar xvf OpenWrt-SDK-sunxi-for-linux-x86_64-gcc-4.6-linaro_uClibc-0.9.33.2.tar.bz2
cd  OpenWrt-SDK-sunxi-for-linux-x86_64-gcc-4.6-linaro_uClibc-0.9.33.2
```

2. 建立软件工作目录

```
cd package
mkdir helloworld
vim Makefile    #这个Makefile可以作为模板
##############################################
# OpenWrt Makefile for helloworld program
#
#
# Most of the variables used here are defined in
# the include directives below. We just need to
# specify a basic description of the package,
# where to build our program, where to find
# the source files, and where to install the
# compiled program on the router.
#
# Be very careful of spacing in this file.
# Indents should be tabs, not spaces, and
# there should be no trailing whitespace in
# lines that are not commented.
#
############################################## 

include $(TOPDIR)/rules.mk 

# Name and release number of this package 
PKG_NAME:=helloworld 
PKG_RELEASE:=1 

# This specifies the directory where we’re going to build the program.
# The root build directory, $(BUILD_DIR), is by default the build_mipsel
# directory in your OpenWrt SDK directory

PKG_BUILD_DIR := $(BUILD_DIR)/$(PKG_NAME)
include $(INCLUDE_DIR)/package.mk

# Specify package information for this program.
# The variables defined here should be self explanatory.
# If you are running Kamikaze, delete the DESCRIPTION
# variable below and uncomment the Kamikaze define
# directive for the description below

define Package/helloworld
    SECTION:=utils
    CATEGORY:=Utilities
    TITLE:=Helloworld — prints a snarky message
endef 

# Uncomment portion below for Kamikaze and delete DESCRIPTION variable above
define Package/helloworld/description
        If you can’t figure out what this program does, you’re probably
        brain-dead and need immediate medical attention.
endef 

# Specify what needs to be done to prepare for building the package.
# In our case, we need to copy the source files to the build directory.
# This is NOT the default.  The default uses the PKG_SOURCE_URL and the
# PKG_SOURCE which is not defined here to download the source from the web.
# In order to just build a simple program that we have just written, it is
# much easier to do it this way. 

define Build/Prepare
    mkdir -p $(PKG_BUILD_DIR)
    $(CP) ./src/* $(PKG_BUILD_DIR)/
endef 

# We do not need to define Build/Configure or Build/Compile directives
# The defaults are appropriate for compiling a simple program such as this one
# Specify where and how to install the program. Since we only have one file,
# the helloworld executable, install it by copying it to the /bin directory on
# the router. The $(1) variable represents the root directory on the router running
# OpenWrt. The $(INSTALL_DIR) variable contains a command to prepare the install
# directory if it does not already exist.  Likewise $(INSTALL_BIN) contains the
# command to copy the binary file from its current location (in our case the build
# directory) to the install directory. 

define Package/helloworld/install
    $(INSTALL_DIR) $(1)/bin
    $(INSTALL_BIN) $(PKG_BUILD_DIR)/helloworld $(1)/bin/
endef 

# This line executes the necessary commands to compile our program.
# The above define directives specify all the information needed, but this
# line calls BuildPackage which in turn actually uses this information to
# build a package.

$(eval $(call BuildPackage,helloworld))
mkdir src
cd src
```

3. 编写自己的软件, 这里以helloworld为例.

```
vim helloworld.c

#include<stdio.h>
int main(void)
{
    printf(“Hell! O’ world, why won’t my code compile?\n\n”); 
    return 0;
}

vim Makefile

# build helloworld executable when user executes "make"
helloworld: helloworld.o 
    $(CC) $(LDFLAGS) helloworld.o -o helloworld

helloworld.o: helloworld.c 
    $(CC) $(CFLAGS) -c helloworld.c 

# remove object files and executable when user executes "make clean"
clean:
    rm *.o helloworld
```

4. 编译软件, 回到SDK根目录下.

```
cd ../../
make V=s
pillar@monster :~/openwrt/trunk/bin/sunxi/OpenWrt-SDK-sunxi-for-linux-x86_64-gcc-4.6-linaro_uClibc-0.9.33.2$ ls bin/sunxi/packages/
helloworld_1_sunxi.ipk  Packages  Packages.gz
```

5. 修改软件源根目录. 如果你不想每次都拷贝, 你可以把软件源的根目录下设置在

```
OpenWrt-SDK-sunxi-for-linux-x86_64-gcc-4.6-linaro_uClibc-0.9.33.2/bin/sunxi/packages 
```

然后你可以在OpenWrt系统里面下载安装.

### Refs

* <http://cnlearn.linksprite.com/?p=2724#tab-1392544656-1-91>

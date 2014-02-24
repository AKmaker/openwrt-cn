
### OpenWrt Customization

OpenWrt is a GNU/Linux distribution targeting router-like embedded devices: it is the perfect choice to use as the "brain" of our Wi-Fi Car.

The device we chose is the [TP-Link WR703N](http://wiki.openwrt.org/toh/tp-link/tl-wr703n) – a pocket-size router with reasonable price tag. However, you may use any OpenWrt-compatible router with a USB interface to do the job.

The WR703N has only 4MB of flash ROM, meaning you must be very careful with your software selection.

Pre-compiled images are available on our [downloads](http://www.open-drone.org/resources) page.

### Getting the Source

```
sudo apt-get install subversion build-essential
svn co svn://svn.openwrt.org/openwrt/trunk/
```

Download and install feeds using feeds script.

```
cd <path to trunk>
./scripts/feeds update -a
./scripts/feeds install -a
```

### Configuring OpenWrt

```
make menuconfig
```

If you get errors with 'make menuconfig', try installing the following packages.

```
sudo apt-get install libncurses5-dev zlib1g-dev gawk unzip git-core
```

The TP-Link WR703N is based on an AR7240 SoC, so chose Atheros AR7xxx/AR9xxx as the target system.

```
Target System  -> Atheros AR7xxx/AR9xxx
Target Profile -> TL-WR703N
```

To support USB storage with the ext4 filesystem, choose the following options.

```
Kernel modules -> Filesystems -> kmod-fs-ext4
Kernel modules -> USB Support -> kmod-usb-storage
```

To support UVC-compatible USB webcams, choose the following options.

```
Kernel modules -> Video Support -> kmod-video-core
Kernel modules -> Video Support -> kmod-video-uvc
```

To support the Arduino USB console, choose the following options.

```
Kernel modules -> USB Support -> kmod-usb-acm
Kernel modules -> USB Support -> kmod-usb-serial
Kernel modules -> USB Support -> kmod-usb-serial-ftdi
```

To mount the USB storage under the root filesystem, choose the following options.

```
Base System->block-mount
```

To add video streaming, choose the following option.

```
Multimedia -> mjpg-streamer
```

Check this to enable setting the speed of serial communication via stty

```
Base system -> busybox -> Coreutils -> stty
```

To add web server function, choose the following option.

```
Network -> Web Servers/Proxies -> nginx
```

### Refs
* <http://www.open-drone.org/openwrt>
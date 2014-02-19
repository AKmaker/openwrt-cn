
The following guide will walk you through the downloading, configuring, and compilation of OpenWRT. The guide makes no assumption about the reader’s previous experience with OpenWRT, or even building from source, so as long as you follow along you should be fine. That said, experience with compiling software and general troubleshooting skills would help make sure the process goes smoothly.

Disclaimer: The final part of this guide will have you install your self-built firmware image onto your router. While the ability to compile and configure your own router firmware is an incredible achievement for the open source community, it’s also somewhat risky. Make sure to familiarize yourself with the firmware recovery options that are specific to your device by visiting the OpenWRT "Table of Hardware", located at: <http://wiki.openwrt.org/toh/start>

### What is OpenWRT?

It’s easy to assume that OpenWRT is "just another router firmware", like the well known **DD-WRT** or **Tomato** firmwares. In fact, up until just a few months ago, that’s what I thought myself. But whereas **DD-WR** aims at being a feature-rich router firmware that will give you an out of the box experience closer to what you would see on a high-end enterprise router, OpenWRT aims to give you a full Linux system that just happens to be running on a router. With OpenWRT installed on your router, you’ll have something that’s much closer to a minimal Linux install on your PC than what you would expect from a router. Out of the box there’s no friendly web UI to set up the router, just the prospect of logging in over SSH and editing text files by hand.

The possibilities for such a system are endless. With a large package repository, an OpenWRT device has access to software you may never have even considered running on a router. You can setup an OpenWRT device to run as a file server, a media extender, a security auditing toolbox, or anything else you could do with a normal Linux system. The only limitation is the processing capability of the specific hardware OpenWRT runs on, which can be anywhere from a cheap router that barely has enough RAM to boot the system to devices which could give some smartphones a run for their money. The individual hardware you chose will depend on many factors (probably not least of which, your budget), but the process described here works the same on any device OpenWRT can or will support.

### Why Build from Source?

While the OpenWRT project makes firmware images available for all of the various devices they support, these aren’t always ideal depending on what your goals are. The stock images from OpenWRT are setup with the assumption that you want to end up with a more or less standard consumer router when you are done with the installation, which may be far from the case. Due to the limited storage space available on these devices, you can’t afford to have your flash storage taken up by things like firewalls and web configuration pages if you are looking to turn an OpenWRT-capable router into a robot controller or a dedicated file server on your network.

From personal experience, the router I use for OpenWRT development has so little storage space left after installation of the stock OpenWRT firmware image that I couldn’t install even a single additional package. For me, I had to build from source just to get any use out of my device.

Even if you have the storage space to burn, learning the ins and outs of OpenWRT by compiling it from source is an excellent way to learn how the system works. If you plan on doing any kind of modification to OpenWRT, it’s definitely in your best interests to dive right in and see how the whole build process functions.

### Requirements

Before we begin, I should make clear that there are some hardware requirements for building a project of this size. While you’ll only need about 300 MB to download the OpenWRT source, the size of the final compiled code can easily get up to few gigabytes, so keep that in mind if your are on a size constrained setup.

You’ll also need a rather powerful machine to work through this build. For my own development I use a 6 core AMD machine with 4 GB of RAM, and it takes around 20 minutes to build a minimal OpenWRT firmware image. On a dual or even single core machine, the build times will likely be into the hours, especially if you are including a lot of software in the image.

### Setting up the Build Environment

To start, you’ll need to prepare your system for building packages from source if it isn’t already, and install Subversion which is used by the OpenWRT developers to distribute the code. On an **APT-based** system (Ubuntu, Mint, etc), the following command should cover all the bases:

```bash
bash$ sudo apt-get install subversion build-essential libncurses5-dev zlib1g-dev gawk flex quilt git-core
```

With those packages installed, you’ll be able to get the latest source code from the OpenWRT developers and begin the process of turning that into an installable image.

At this point you’ll need to make a determination on which version of OpenWRT you want to run. There are currently two branches, the stable branch called "Backfire", and the development branch which is referred to as "trunk". While some devices are well supported via **Backfire**, newer devices will require you use **trunk** as support for them hasn’t always hit the point where the developers are ready to call it stable. For the purposes of this guide we will be going along with **trunk** as it has the most hardware support, but the steps will be the same if you chose to download **Backfire** instead.

To begin the download of OpenWRT, first decide where you want to store and build the source. Some like to do this out of **/usr/src**, but any directory will work. For simplicities sake, let’s build it out of our **/home** directory. Run the following commands to download OpenWRT and get yourself into the source tree:

```bash
bash$ mkdir ~/OpenWRT
bash$ cd OpenWRT
bash$ svn co svn://svn.openwrt.org/openwrt/trunk/
bash$ cd trunk
```

### Optional Packages

The previous commands got the core of OpenWRT downloaded, and you now have everything you need to build a minimal firmware image for your router. But depending on what you want to do, there might be additional software outside of the stock OpenWRT tree that you want to include in your final firmware image. OpenWRT includes a tool called "feeds" which helps find, update, and install these additional packages, which you should run before going any farther if you wish to include any of them. To start using the "feeds" tool, you first need to tell it which repositories you want to include. This is done through a file in the root of the OpenWRT source called "feeds.conf.default". Opening it up in your text editor of choice should present you with the following:

```
src-svn packages svn://svn.openwrt.org/openwrt/packages
src-svn xwrt http://x-wrt.googlecode.com/svn/trunk/package
src-svn luci http://svn.luci.subsignal.org/luci/trunk/contrib/package
#src-svn phone svn://svn.openwrt.org/openwrt/feeds/phone
#src-svn efl svn://svn.openwrt.org/openwrt/feeds/efl
#src-svn xorg svn://svn.openwrt.org/openwrt/feeds/xorg
#src-svn desktop svn://svn.openwrt.org/openwrt/feeds/desktop
#src-svn xfce svn://svn.openwrt.org/openwrt/feeds/xfce
#src-svn lxde svn://svn.openwrt.org/openwrt/feeds/lxde
#src-link custom /usr/src/openwrt/custom-feed
```

Here we can see that only the first three repositories are enabled, as everything else is commented out with the "# character. For most tasks the first three repositories should be sufficient, but if you are looking to get into the more advanced territories, like installing X and using graphical tools on it, you’ll need to enable the appropriate repositories in this file by removing the "# signs before each line.

Now that we’ve enabled the repositories we want to use, the next step is to download the list of available packages by running the following command from within the OpenWRT source tree:

```bash
bash$ scripts/feeds update -a
```

You’ll then see a very long list of packages being downloaded and updated, after which you’ll be dropped back to the shell. With the updated list of packages in hand, you can now search for any install anything you may want which isn’t included in OpenWRT. For example, if you were looking for Bluetooth related packages you could run the following:

```
bash$ scripts/feeds search Bluetooth
Search results in feed 'packages':
anyremote                	A bluetooth remote control app
bemusedlinuxserver       	Bemused linux server
bluez-hcidump            	Bluetooth packet analyzer
bluez-libs               	Bluetooth library
bluez-utils              	Bluetooth utilities
miax                     	A console iax (asterisk) client
python-bluez             	Python wrapper for the BlueZ Bluetooth stack
```

Let’s say you then wanted to install **bluez-utils** and **bluez-libs**, you could run the following command:

```bash
bash$ scripts/feeds install bluez-libs bluez-utils
Collecting package info: done
Collecting target info: done
Installing package 'bluez-libs'
Installing package 'gettext'
Installing package 'libiconv'
Installing package 'bluez-utils'
Installing package 'dbus'
Installing package 'expat'
Installing package 'gettext-full'
Installing package 'libiconv-full'
```

Note that "feeds" is smart enough to resolve dependencies for those two packages, and has therefore installed quite a few other packages which will be required to use Bluetooth on your final OpenWRT firmware.

### Configuring the Build

With the OpenWRT source (and any additional packages) installed, you can move on to configuring the actual build so it is tailored to both your hardware and the tasks you want to perform. The menu system for configuring OpenWRT is identical to what is used to configure and compile the Linux kernel itself, so if you’ve ever built a custom kernel you should feel right at home here.

To start the process, run the command:

```bash
bash$ make menuconfig
```

After a few seconds you should see the following screen:

![Main configuration screen for OpenWRT]()

This is the main screen for configuring OpenWRT, here you’ll see an overview of what hardware you’ll be building for and what software and libraries will be built into the image. The top of the window shows instructions on how to interact with the menu selections, and you can select "Help" to get more information about most menu selections.

The first part of configuring OpenWRT is to select what hardware you’ll be building with with the top two items:

1. **Target System**: The core architecture for your device, check OpenWRT Wiki if you don’t know what it is.

2. **Target Profile**: Specific router make/model. Not all models have specific build profiles, otherwise, the default **generic** profile will probably work.

Going down the list, the next couple options should be:

1. **Build the OpenWRT Image Builder**

2. **Build the OpenWRT SDK**

3. **Build the OpenWRT based Toolchain**

All three of these options are primarily designed for distribution by the OpenWRT developers, as they will create packages which allow the user to create firmware images without having to compile OpenWRT, and build packages for additional software. Since we are already building all of OpenWRT from source already, these packages won’t help us; they can be left disabled.
 
The next option that will be of interest to us is "Base system", which will let you pick individual software packages you wish to include in your firmware:

![Base system configuration for OpenWRT]()

After you’re done selecting which of the **base packages** you need, back out of that screen and select "Libraries" from the main menu. That will present you with the same type of menu as "Base system", but this time you’ll be going through the **system libraries**.

![Library configuration for OpenWRT]()

Note the listing for "bluez-libs", this is showing up because I brought that package in using the "feeds" tool earlier. Any optional packages you downloaded into the OpenWRT build directory in the previous step will need to be selected for inclusion, as installing them via "feeds" simply grabs the source code and Makefiles, it doesn’t actually put them into the firmware image.

With your libraries selected, you can go back up to the top menu and chose the next category on the list. Depending on what (if any) optional packages you included earlier, you may see different categories listed. The process with all of the categories is the same, go into them, select the packages you want to include or exclude from the final image, and then back out to the main menu to proceed down to the next selection.

When finished going through all the categories of software, you can back out of the menu system entirely. Make sure to confirm that you want to save your OpenWRT configuration when it asks you.

### Choose Wisely

While the OpenWRT build environment does do some basic dependency checking when you select a package for inclusion in your firmware, it’s still very possible to create a firmware image that simply won’t boot up due to missing software. In fact, some might argue it’s very easy to do just that.

Before deselecting too many items in the build configuration, make sure you have a pretty strong handle on basic Linux concepts and what makes the system run. If you have any doubts about what a particular package does in OpenWRT or Linux in general, it’s better to do a quick search for the package name online than just disabling it and seeing what happens.
 
On the other hand, the devices that OpenWRT runs on have an extremely limited amount of storage capacity. The router I personally do my OpenWRT work on only has 4 MB of flash. A bare bones OpenWRT image weighs in at around 3 MB, so you can see how there isn’t a lot of leg room on my hardware to start adding in every package that seems interesting. When selecting packages for your firmware, keep in mind the storage limitations of your device. Some packages are simply too large to fit in flash and may therefore rely on an external storage device, which in turn relies on your hardware having some way of connecting a storage device to it. It’s just as easy to select too many packages as it is too few.

### Compiling the Image

With all of your packages selected and your configuration saved, you’re ready to build your firmware.
On single core machines you would simply run:

```bash
bash$ make
```

On dual or more core machines, you can break the build process up to take advantage of multiple processors. The rule of thumb is to use the number of processors plus 1 for how many concurrent jobs you want to use. So on a 6 core machine you would run:

```bash
bash$ make -j 7
```

In either event, the next step is…to wait. The OpenWRT build process is rather lengthy, so it could be a while until this process completes. If the build ends with an error, you can re-run make with more verbose output by running the following command:

```bash
bash$ make V=99
```

### Installing the Image

Hopefully the build process went well for you and didn’t take a ridiculous amount of time to complete. If you got to the end and there were no errors, then you should have firmware images for your hardware created under the **/bin** directory of the OpenWRT source. For example, when building for my **TP-Link MR3220**, I get the following files:

```bash
./bin/ar71xx/openwrt-ar71xx-tl-mr3220-v1-squashfs-factory.bin
./bin/ar71xx/openwrt-ar71xx-tl-mr3220-v1-squashfs-sysupgrade.bin
```

The first file, **factory.bin**, is for flashing over the original firmware your device came with. The second file, **sysupgrade.bin**, is for upgrading an existing OpenWRT installation. Don’t try to flash a **sysupgrade.bin** file over your stock firmware, as the results can be unpredictable.

Follow the device-specific instructions from the OpenWRT Wiki to find out the exact process required to replace your device’s stock firmware with this brand new image. Some devices allow you to install any image you like through their web configuration, other devices may require more exotic installation methods.

### Where do we go from here?

If everything has gone as planned to this point, you should have a custom OpenWRT build running on your hardware. Poke around it a bit to make sure everything you wanted was installed and is working as expected. Keep an eye on "The Powerbase" for a future guide on how to setup your new OpenWRT device, as well as some interesting things you can do with that miniature Linux box.

### Ref

* <http://www.thepowerbase.com/2012/01/openwrt-build-guide-start-to-finish/>



[TOC]

### Introduction

I'm writing this document because I found the documentation available on the [OpenWrt wiki](http://wiki.openwrt.org) difficult to follow. I found myself in the position of wanting to compile a very simple program that I could run on my OpenWrt router. I ended up spending hours wading through the frustratingly incomplete documentation on the wiki, going through dozens of forum posts, and conducting extensive trial-and-error before my code would compile. I especially appreciated the examples on the wiki that contain the warning: "Note this Makefile is provided as an example only; it will not compile." If something doesn't work, it isn't a very good example, is it? Here, then, is a (hopefully) more straightforward guide to building a program for OpenWrt. I found that existing documentation focuses more on porting existing, complicated programs to OpenWrt. My intention is to focus on getting a small, very simple, home-grown application running on OpenWrt. My goal is to explain this in as simple and complete a manner as possible, explaining each and every step necessary to write and compile a program that will run on OpenWrt. The process is actually very simple and straightforward – provided you know what you're doing. For the purposes of this tutorial I'm going to assume you have a development box running linux and a router running OpenWrt. I will also assume you are at least somewhat familiar with C/C++ and standard Unix Makefiles.
 
The code for the examples in this tutorial can be downloaded from [here](). The example from the first part of the tutorial is in the **openwrt-programming-examples/c** directory and the example for the White Russian portion of the second part is in the **openwrt-programming-examples/c++** directory. The modified Kamikaze SDK Makefile referred to in part III is also included in this zip file.
 
### Part I: A Simple Program in C -- 8/23/2007

First, we're going to need to write the code for the program itself and get it compiling on our local linux machine. Let's write a simple "hello world" program that we want to run on the router:

~/helloworld/src/helloworld.c:

```C
/****************
* Helloworld.c
* The most simplistic C program ever written. 
* An epileptic monkey on crack could write this code.
*****************/

#include <stdio.h>
int main(void)
{
     printf("Hell! O' world, why won't my code compile?\n\n");
     return 0; 
}
```

Alright, we have our code. Note the location of this file. Make a **helloworld** directory and then a **src** subdirectory. Place the code in the **src** subdirectory. Now, let's write a standard Unix Makefile to compile this code for us: 

```Makefile
# build helloworld executable when user executes "make" 

helloworld: helloworld.o
     $(CC) $(LDFLAGS) helloworld.o -o helloworld

helloworld.o: helloworld.c
     $(CC) $(CFLAGS) -c helloworld.c

# remove object files and executable when user executes "make clean"
clean:
     rm *.o helloworld
```

Notice that instead of hard-coding “gcc” in the makefile to compile the program, we use a variable that holds the C compiler, $(CC). If you're compiling a c++ program you would use $(CXX) instead of $(CC) and $(CXXFLAGS) instead of $(CFLAGS). The use of the compiler variable is not necessary to compile the code locally, but in order to compile the code for OpenWRT it is critical because we won't be using the standard gcc compiler. Place the makefile in the same **src** directory our code is in. We can now go to the **src** directory, type “make” and the program should compile. You can run it by typing "./helloworld".

Be sure to run “make clean” to remove the compiled binary and object files after you have successfully tested the program. We're going to build the source files in this directory for the router, and if the old object file isn't deleted, it will be used instead of the new one we need to generate that is compatible with OpenWrt. 

```bash
mockingbird@linuxbox:~/helloworld/src$ make
mockingbird@linuxbox:~/helloworld/src$ ./helloworld
Hell! O' world, why won't my code compile?
mockingbird@linuxbox:~/helloworld/src$ make clean
rm *.o helloworld 
mockingbird@linuxbox:~/helloworld/src$
```

So far this should be a review on how to write simple C programs and how to use Makefiles. Now comes the tricky part, compiling the code so that it will run on our router. The router uses a distinctly different architecture than our linux development box. Because there isn't enough memory/disk space on the router to install a compiler and compile the code natively, we need to “cross-compile” the code on our development box for use on the router. To do this we need a special compiler and development environment called the OpenWRT SDK. You can download the SDK from <http://downloads.openwrt.org> The SDK varies depending on the architecture of your development box, the architecture of your router and the version/release of OpenWrt your router is running. I currently have whiterussian v0.9 installed on my Linksys WRT54G router, and my development box is an i686, so the SDK I use is [this one](http://downloads.openwrt.org/whiterussian/0.9/OpenWrt-SDK-Linux-i686-1.tar.bz2). Extract the SDK files from the downloaded archive, and enter the SDK directory, which should have the same name as the tar.bz2 file (in my case OpenWrt-SDK-Linux-i686-1).

```bash
mockingbird@linuxbox:~$ tar xfj OpenWrt-SDK-Linux-i686-1.tar.bz2
mockingbird@linuxbox:~$ cd OpenWrt-SDK-Linux-i686-1
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ ls
dl  docs  examples  include  Makefile  package  README.SDK  rules.mk  scripts  
staging_dir_mipsel
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ 
```

Our goal is to build a package for OpenWrt using the source we already have. When you execute the "make" command in the SDK directory, the SDK will compile all properly configured packages in the **package** subdirectory under the SDK directory. The next step (and the trickiest) is to properly configure our code so that the SDK will build it. First, copy the **helloworld** directory we made earlier into the **package** subdirectory of the SDK:

```bash
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ cp -r ~/helloworld package
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$
```

In order to tell the OpenWrt SDK how to build our program we need to create a special Makefile in the **helloworld** directory, above the src directory which contains our conventional makefile. Writing this file is 90% of the work involved in compiling our program for OpenWrt. Below is an OpenWrt makefile for building the helloworld program. Each section is heavily commented so that it should be fairly clear what is going on: 

~/OpenWrt-SDK-Linux-i686-1/package/helloworld/Makefile:

```Makefile
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


# This specifies the directory where we're going to build the program.  
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
	TITLE:=Helloworld -- prints a snarky message
	DESCRIPTION:=\
	If you can't figure out what this program does, \\\
	you're probably brain-dead and need immediate \\\
	medical attention.
endef


# Uncomment portion below for Kamikaze and delete DESCRIPTION variable above
#define Package/helloworld/description
#	If you can't figure out what this program does, you're probably
#	brain-dead and need immediate medical attention.
#endef



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

```

As indicated, most OpenWrt make files specify how to download the source of an application from a URL, and most documentation assumes that you want to do this. However, if you're building your own application from scratch it doesn't make sense to download from a URL. It's much simpler to have the source locally and use the Build/Prepare section to copy the source to the build directory, as shown above. Also, be very careful of spacing in the Makefile. The indentation under the define sections should be tabs, not spaces and there should be no whitespace at the end of lines that are not comments. The trailing whitespace can be a problem when variables are being defined, as the compiler may think there is a space at the end of a directory name. If we're copying something to dir_with_trailing_space/subdir the copy command may be executed as "cp my.file dir_with_trailing_space /subdir". Not only don't you want anything in /subdir, you probably don't have permission to create it and write to it. 

Also note that the syntax for specifying the description is slightly different between the White Russian and Kamikaze versions of the SDK. The build system in Kamikaze requires that you specify the description using a seperate section so delete the description variable in the Package/helloworld section and uncomment the kamikaze description section above as indicated. 

Now we're all set to compile the helloworld package. Go to the root SDK directory (if you're not already there) and type “make V=99” The “V=99” option is optional, but it is useful for debugging as it instructs the compiler to be “verbose” and output all the details of what it is doing.

```bash
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ make V=99
make package/compile
make[1]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1'
Collecting package info...
make -C package compile SDK=1
make[2]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[2]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[2]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make -j1 compile-targets
make[3]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make -C helloworld compile
make[4]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package/he
lloworld'
CFLAGS="-Os -pipe -mips32 -mtune=mips32 -funit-at-a-time -I/home/mockingbird/OpenWr
t-SDK-Linux-i686-1/staging_dir_mipsel/usr/include -I/home/mockingbird/OpenWrt-SDK-L
inux-i686-1/staging_dir_mipsel/include " LDFLAGS="-L/home/mockingbird/OpenWrt-SDK-L
inux-i686-1/staging_dir_mipsel/usr/lib 
-L/home/mockingbird/OpenWrt-SDK-Linux-i686-1/staging_dir_mipsel/lib " make -C /home
/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld AR=mipsel-linux-uclib
c-ar AS="mipsel-linux-uclibc-gcc -c -Os -pipe -mips32 -mtune=mips32 -funit-at-a-tim
e" LD=mipsel-linux-uclibc-ld NM=mipsel-linux-uclibc-nm CC="mipsel-linux-uclibc-gcc"
GCC="mipsel-linux-uclibc-gcc" CXX=mipsel-linux-uclibc-g++ RANLIB=mipsel-linux-uclib
c-ranlib STRIP=mipsel-linux-uclibc-strip OBJCOPY=mipsel-linux-uclibc-objcopy CROSS=
"mipsel-linux-uclibc-" CXXFLAGS="-Os -pipe -mips32 -mtune=mips32 -funit-at-a-time -
I/home/mockingbird/OpenWrt-SDK-Linux-i686-1/staging_dir_mipsel/usr/include -I/home/
mockingbird/OpenWrt-SDK-Linux-i686-1/staging_dir_mipsel/include " ARCH="mipsel" ;
make[5]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipse
l/helloworld'
make[5]: `helloworld' is up to date.
make[5]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel
/helloworld'
touch /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/.built
install -d -m0755 /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworl
d/ipkg/helloworld/bin
install -m0755 /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/h
elloworld /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/ipkg/h
elloworld/bin/
mkdir -p /home/mockingbird/OpenWrt-SDK-Linux-i686-1/bin/packages
find /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/ipkg/hellow
orld -name CVS | xargs rm -rf
find /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/ipkg/hellow
orld -name .svn | xargs rm -rf
find /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/ipkg/hellow
orld -name '.#*' | xargs rm -f
STRIP="/home/mockingbird/OpenWrt-SDK-Linux-i686-1/staging_dir_mipsel/bin/sstrip" ST
RIP_KMOD="mipsel-linux-uclibc-strip --strip-unneeded --remove-section=.comment" /ho
me/mockingbird/OpenWrt-SDK-Linux-i686-1/scripts/rstrip.sh /home/mockingbird/OpenWrt
-SDK-Linux-i686-1/build_mipsel/helloworld/ipkg/helloworld
rstrip.sh: /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/helloworld/ipkg/
helloworld/bin/helloworld:executable
ipkg-build -c -o 0 -g 0 /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/hel
loworld/ipkg/helloworld /home/mockingbird/OpenWrt-SDK-Linux-i686-1/bin/packages
Packaged contents of /home/mockingbird/OpenWrt-SDK-Linux-i686-1/build_mipsel/hellow
orld/ipkg/helloworld into /home/mockingbird/OpenWrt-SDK-Linux-i686-1/bin/packages/h
elloworld_1_mipsel.ipk
make[4]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package/hell
oworld'
make[3]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[2]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[1]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1'
( \
        cd package; \
        find . -maxdepth 2 -name Config.in | \
                sed -e 's,/Config.in,,g' | \
                xargs -r -n1 make compile -C; \
)
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$
```

It compiled! The new package, helloworld_1_mipsel.ipk, is now located in the **bin/packages** subdirectory of the root SDK directory. 

```bash
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ cd bin/packages
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1/bin/packages$ ls
helloworld_1_mipsel.ipk
mockingbird@linuxbox:~OpenWrt-SDK-Linux-i686-1/bin/packages$
```

This file is a ipk file which is used by the ipkg (itsy package management) system. Ipkg is a package management system for embedded devices, where space is an issue. Let's copy this package onto the router, which is located at 192.168.1.1 on my network. 

```bash
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1/bin/packages$ scp helloworld_1_mip
sel.ipk root@192.168.1.1:
root@192.168.1.1's password: 
helloworld_1_mipsel.ipk                              100% 1875     1.8KB/s   00:00
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1/bin/packages$
```

Now, ssh into the router. We just copied the package to root's home directory so we are finally ready to install our program. In root's home directory, (where we end up immediately after connecting to the router via ssh) type “ipkg install helloworld_1_mipsel.ipk” [NOTE: on OpenWrt 8.09 and later you need to run opkg install helloworld_1_mipsel.ipk, as ipkg has been replaced with a new, improved utility called opkg] and the ipkg system will do the rest.

```bash
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1/bin/packages$ ssh root@192.168.1.1
root@192.168.1.1's password: 


BusyBox v1.00 (2007.01.30-11:42+0000) Built-in shell (ash)
Enter 'help' for a list of built-in commands.

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 WHITE RUSSIAN (0.9) -------------------------------
  * 2 oz Vodka   Mix the Vodka and Kahlua together
  * 1 oz Kahlua  over ice, then float the cream or
  * 1/2oz cream  milk on the top.
 ---------------------------------------------------
root@OpenWrt:~# ls
TZ                       ip-up                    resolv.conf              spool
dhcp.leases              log                      resolv.conf.auto         usr
helloworld_1_mipsel.ipk  net                      run
root@OpenWrt:~# ipkg install helloworld_1_mipsel.ipk
Installing helloworld (1) to root...
Configuring helloworld
Successfully terminated.
root@OpenWrt:~# 
```

The executable has now been installed into the /bin directory on the router, per our instructions in the OpenWrt Makefile listed above. So, all we have to do to run the program is type “helloworld” at the prompt. Note that because the executable has been installed to the /bin directory you should be able to execute the program no matter what directory you are in on the router. 

```bash
root@OpenWrt:~# helloworld
Hell! O' world, why won't my code compile?

root@OpenWrt:~#
```

### Part II: C++ and the Standard Template Library (STL) -- 10/10/2007

If you only want to compile C programs, the first part of my tutorial should be enough to get you started. However, if you want to use C++ there is another issue you're likely to run into. Let's say we want to compile the following C++ program for OpenWrt:

~/OpenWrt-SDK-Linux-i686-1/package/helloworld/src/helloworld.cpp:

```CPP
/****************
* Helloworld.cpp
*****************/
#include <iostream>
#include <string>

using namespace std;

int main()
{
	string s = "Hell! O' world, why won't my c++ code run?\n\n";
	cout << s;
	return 0;
}
```

So, let's do exactly as we did before and see what happens. We just have to update the compiler flags in the Makefile to indicate that we're using a C++ compiler instead of a C compiler, right?

~/OpenWrt-SDK-Linux-i686-1/package/helloworld/src/Makefile: **NOTE: This code will compile but not run, further modifications are needed which are described below**

```Makefile
# build helloworld executable when user executes "make"
helloworld: helloworld.o
	$(CXX) $(LDFLAGS) helloworld.o -o helloworld
helloworld.o: helloworld.cpp
	$(CXX) $(CXXFLAGS) -c helloworld.cpp

# remove object files and executable when user executes "make clean"
clean:
	rm *.o helloworld

```

If you use the exact same OpenWrt Makefile as before, this program will compile just fine. It will even install properly when you copy the helloworld_1_mipsel.ipk file to the router and type "ipkg install helloworld_1_mipsel.ipk" However, watch what happens if you try to run the program:

```bash
root@OpenWrt:~# helloworld
helloworld: can't load library 'libstdc++.so.6'
root@OpenWrt:~#
```

So, what went wrong? The problem is that this program uses strings and iostreams which are a feature of the C++ standard template library (STL). However, because memory is so critical in an embedded application like OpenWrt, the standard template library is not available by default.

What needs to be done to fix the problem depends on which version of OpenWrt you are running. If you are running *Kamikaze*, the problem is much easier to fix than if you are running *White Russian*. If you are running *Kamikaze*, you only need to install the **libstdcpp library**. If you are running *White Russian*, as I am, you must install the **uclibcxx** library, as well as make certain changes to the Makefiles to use this library, which is a special implementation of the standard library for embedded devices. If you are running *Kamikaze* follow the first part of the following instructions to install **libstdcpp** instead of **uclibcxx**, and ignore the changes to the Makefiles. If you are running White Russian, you can follow these directions exactly. Note that example files for this portion of the tutorial assume you are running *White Russian*, and will not work with *Kamikaze*.

Before we adjust our Makefiles to use to this alternate library, let's first make sure that it's installed on the router so that it is available to link to. Log into your router. Then, make sure your list of packages is up to date by using the "ipkg update(opkg updage)" command:

```bash
root@OpenWrt:~# ipkg update
Downloading http://downloads.openwrt.org/backports/0.9/Packages
Updated list of available packages in /usr/lib/ipkg/lists/0.9-backports
Downloading http://download2.berlios.de/pub/xwrt/packages/Packages
Updated list of available packages in /usr/lib/ipkg/lists/X-Wrt
Downloading http://downloads.openwrt.org/whiterussian/packages/Packages
Updated list of available packages in /usr/lib/ipkg/lists/whiterussian
Downloading http://downloads.openwrt.org/whiterussian/packages/non-free/Packages
Updated list of available packages in /usr/lib/ipkg/lists/non-free
Successfully terminated.
root@OpenWrt:~# 
```

Once you have updated your package list, install **uclibcxx** by typing "ipkg install uclibcxx" If you do not have **uclibcxx** installed this should install it:

```
root@OpenWrt:~# ipkg install uclibcxx
Installing uclibcxx (0.2.1-1) to root...
Downloading http://downloads.openwrt.org/backports/0.9/uclibcxx_0.2.1-1_mipsel.ipk
Configuring uclibcxx
Successfully terminated.
root@OpenWrt:~# 
```

Otherwise, if the library is already installed, you'll see this confirmation: 

```bash
root@OpenWrt:~# ipkg install uclibcxx
Package uclibcxx (0.2.1-1) installed in root is up to date.
Nothing to be done
Successfully terminated.
root@OpenWrt:~# 
```

Now it's time to modify the Makefiles. First, the one in the src directory. Only a small change is necessary here, to the line where we link the objects together. We need to be able to tell the linker to link to another library, and the way we do that is by defining a variable called $(LIBS). The variable will be defined in the special OpenWrt Makefile, but will be used here, so we need to add it to the end of the line which specifies how to do the linking:

```Makefile
# build helloworld executable when user executes "make"
helloworld: helloworld.o
	$(CXX) $(LDFLAGS) helloworld.o -o helloworld $(LIBS)
helloworld.o: helloworld.cpp
	$(CXX) $(CXXFLAGS) -c helloworld.cpp

# remove object files and executable when user executes "make clean"
clean:
	rm *.o helloworld
```

The OpenWrt Makefile needs a slightly more complex modification. Recall the comment I placed in the OpenWrt Makefile we used before, "We do not need to define Build/Configure or Build/Compile directives. The defaults are appropriate for compiling a simple program such as this one." Well, guess what? In order to link to **uclibcxx** we need to customize how we're going to compile the program, and we're going to do this in the **Build/Compile** directive. Before we re-define this directive to suit our purposes, let's see what the default looks like:

```Makefile
define Build/Compile/Default
	CFLAGS="$(TARGET_CFLAGS) $(EXTRA_CPPFLAGS) " \
	LDFLAGS="$(EXTRA_LDFLAGS) " \
	$(MAKE) -C $(PKG_BUILD_DIR) \
		$(TARGET_CONFIGURE_OPTS) \
		CROSS="$(TARGET_CROSS)" \
		CXXFLAGS="$(TARGET_CFLAGS) $(EXTRA_CPPFLAGS) " \
		ARCH="$(ARCH)" \
		$(1);
endef
```

This code specifies the make command (the third line) and a bunch of flags and parameters for compilation. In order to link to **uclibcxx** or, **uClibc++** as it is known to the linker, we add the definition of the $(LIBS) variable as "-nodefaultlibs -lgcc -lc -luClibc++" Also, we need to specify "-nostdinc++"" in the compiler flags to tell the compiler that c++ standard template library functions and data structures will be linked to in specified external libraries and not the standard libraries: 

~/OpenWrt-SDK-Linux-i686-1/package/helloworld/Makefile:

```Makefile
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


# This specifies the directory where we're going to build the program.  
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
	TITLE:=Helloworld -- prints a snarky message
	DESCRIPTION:=\
	If you can't figure out what this program does, \\\
	you're probably brain-dead and need immediate \\\
	medical attention.
endef


# Uncomment portion below for Kamikaze and delete DESCRIPTION variable above
#define Package/helloworld/description
#	If you can't figure out what this program does, you're probably
#	brain-dead and need immediate medical attention.
#endef


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


#########################################################################################
# The Build/Compile directive needs to be specified in order to customize compilation
# and linking of our program.  We need to link to uClibc++ and to specify that we 
# do NOT want to link to the standard template library.
#
# To do this we define the LIBS variable.  To prevent linking to the standard libraries we 
# add "-nodefaultlibs" to the $(LIBS) variable and then specify "-lgcc -lc" to ensure that 
# there are no unresolved references to internal GCC library subroutines. Finally 
# "-luClibc++" to link to the  uClibc++ library.  Note the capital C in that flag, as
# this flag is case sensitive.  Also, we need to specify "-nostdinc++" 
# in the compiler flags to tell the compiler that c++ standard template library functions
# and data structures will be linked to in specified external libraries and not the 
# standard libraries.
#########################################################################################
define Build/Compile
	$(MAKE) -C $(PKG_BUILD_DIR) \
		LIBS="-nodefaultlibs -lgcc -lc -luClibc++" \
		LDFLAGS="$(EXTRA_LDFLAGS)" \
		CXXFLAGS="$(TARGET_CFLAGS) $(EXTRA_CPPFLAGS) -nostdinc++" \
		$(TARGET_CONFIGURE_OPTS) \
		CROSS="$(TARGET_CROSS)" \
		ARCH="$(ARCH)" \
		$(1);
endef


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
```

That's it! That modification should be sufficient to enable linking to **uClibc++**. Compile it as before, copy the .ipk file to the router, install it with ipkg and run it: 

```bash
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ make clean
rm -rf build_* bin
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ make
make package/compile
make[1]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1'
Collecting package info...
make -C package compile SDK=1
make[2]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[2]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[2]: Entering directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[4] -C package compile-targets
make[5] -C package/helloworld compile
make[2]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1/package'
make[1]: Leaving directory `/home/mockingbird/OpenWrt-SDK-Linux-i686-1'
( \
        cd package; \
        find . -maxdepth 2 -name Config.in | \
                sed -e 's,/Config.in,,g' | \
                xargs -r -n1 make compile -C; \
)
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ scp bin/packages/helloworld_1_m
ipsel.ipk root@192.168.1.1:
root@192.168.1.1's password: 
helloworld_1_mipsel.ipk                            100% 2570     2.5KB/s   00:00
mockingbird@linuxbox:~/OpenWrt-SDK-Linux-i686-1$ ssh root@192.168.1.1
root@192.168.1.1's password: 


BusyBox v1.00 (2007.01.30-11:42+0000) Built-in shell (ash)
Enter 'help' for a list of built-in commands.

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 WHITE RUSSIAN (0.9) -------------------------------
  * 2 oz Vodka   Mix the Vodka and Kahlua together
  * 1 oz Kahlua  over ice, then float the cream or
  * 1/2oz cream  milk on the top.
 ---------------------------------------------------
root@OpenWrt:~# ipkg install helloworld_1_mipsel.ipk
Installing helloworld (1) to root...
Configuring helloworld
Successfully terminated.
root@OpenWrt:~# helloworld
Hell!  Why won't my c++ code run?
root@OpenWrt:~# 
```

It works! Our Jedi programming skills have prevailed! 

Once again, please do **not** contact me about any issues you might have with your own programs. If,however, you think there is an error in one of my examples or that I have omitted a critical detail, please contact me and I will try to resolve the issue as soon as possible.

### PART III: Building and Using the Kamikaze SDK -- 12/9/2007

The OpenWrt SDK for Kamikaze is available only for 64 bit processors. If you're one of us mere mortals who posesses only one of those antique 32 bit processors, you have a problem. You can sometimes get away with using the White Russian SDK if you only need your program to run on the MIPS architecture, but the new architectures supported only by Kamikaze require you to use a Kamikaze SDK. The solution is to compile the SDK yourself. This is actually fairly simple, provided you have a couple of gigabytes of disk space and an hour or two to spare.

First, download the source from here. Note, that the preceeding link references Kamikaze version 7.09, which is the latest version as I write this. You may wish to check downloads.openwrt.org/kamikaze to see if there is a more recent version. Once you've dowloaded the source, extract the files and enter the Kamikaze build directory: 

```bash
ockingbird@linuxbox:~$ tar xjf kamikaze_7.09.tar.bz2
mockingbird@linuxbox:~$ cd kamikaze_7.09
mockingbird@linuxbox:~/kamikaze_7.09$ ls
BSDmakefile  docs     LICENSE   package  rules.mk  target     tools
Config.in    include  Makefile  README   scripts   toolchain
mockingbird@linuxbox:~/kamikaze_7.09$ 
```

In order to specify the architecture of the router and to specify that we want to build the SDK, we need to run "make menuconfig".

```bash
mockingbird@linuxbox:~/kamikaze_7.09$ make menuconfig
Checking 'working-make'... ok.
Checking 'case-sensitive-fs'... ok.
Checking 'working-gcc'... ok.
Checking 'working-g++'... ok.
Checking 'ncurses'... ok.
Checking 'zlib'... ok.
Checking 'gawk'... ok.
Checking 'bison'... ok.
Checking 'flex'... ok.
Checking 'unzip'... ok.
Checking 'bzip2'... ok.
Checking 'patch'... ok.
Checking 'perl'... ok.
Checking 'wget'... ok.
Checking 'gnutar'... ok.
Checking 'autoconf'... ok.
Checking 'non-root'... ok.
Collecting target info: done
Collecting package info: package/base-filessvn: '.' is not a working copy
Collecting package info: done
```

You will see the following screen. Select the architecture of your router on the top "Target System" line, and then move down and select the 'Build the OpenWrt SDK'. Exit, saving your settings: 

```
       Target System (Broadcom BCM947xx/953xx [2.4])  --->;             
       Target Profile (Generic, Broadcom WiFi (default))  --->;         
       Target Images  --->;                                             
   [ ] Select all packages by default (NEW)                           
   [ ] Advanced configuration options (for developers) (NEW)  --->;    
   [ ] Build the OpenWrt Image Builder (NEW)                          
   [*] Build the OpenWrt SDK                                         
   [ ] Image configuration (NEW)  --->                                    
       Base system  --->                                                  
       Utilities  --->                                                    

                      < Select >    < Exit >    < Help >                 
```

Now, make sure you have at least 1.5 GB of disk space and a good book (I recommend [this](http://search.barnesandnoble.com/booksearch/imageviewer.asp?ean=9780553103540), [this](http://search.barnesandnoble.com/booksearch/isbnInquiry.asp?EAN=9780060013134) or [this](http://search.barnesandnoble.com/booksearch/isbnInquiry.asp?EAN=9780671746728)) because this step is going to take a while. (~90 minutes on my 2.5 Ghz machine with 1 GB of RAM). Just run make:

```bash
  mockingbird@linuxbox:~/kamikaze_7.09$ make
  make[2] -C /home/eric/kamikaze_7.09 tools/install 
  make[3] -C tools install 
  make[4] -C tools/sed prepare 
  make[4] -C tools/sed compile 
  make[4] -C tools/sed install 
  make[4] -C tools/sstrip prepare 
  make[4] -C tools/sstrip compile 
  make[4] -C tools/sstrip install 
  make[4] -C tools/ipkg-utils prepare 
  make[4] -C tools/ipkg-utils compile 
  make[4] -C tools/ipkg-utils install 
  make[4] -C tools/ext2fs prepare 
  make[4] -C tools/ext2fs compile 
  make[4] -C tools/ext2fs install 
  make[4] -C tools/lzma prepare 
  make[4] -C tools/lzma compile 
  make[4] -C tools/lzma install 
  make[4] -C tools/squashfs prepare 
  make[4] -C tools/squashfs compile 
  make[4] -C tools/squashfs install 
  make[4] -C tools/mtd-utils prepare 
  make[4] -C tools/mtd-utils compile 
  make[4] -C tools/mtd-utils install 
  make[4] -C tools/mkimage prepare 
  make[4] -C tools/mkimage compile 
  make[4] -C tools/mkimage install 
  make[4] -C tools/firmware-utils prepare 
  make[4] -C tools/firmware-utils compile 
  make[4] -C tools/firmware-utils install 
  make[4] -C tools/patch-cmdline prepare 
  make[4] -C tools/patch-cmdline compile 
  make[4] -C tools/patch-cmdline install 
  make[2] -C /home/eric/kamikaze_7.09 toolchain/install 
  make[3] -C toolchain install 
  make[4] -C toolchain/kernel-headers prepare 
  make[4] -C toolchain/kernel-headers compile 
  make[4] -C toolchain/kernel-headers install 
  make[4] -C toolchain/uClibc prepare 
  make[4] -C toolchain/binutils prepare 
  make[4] -C toolchain/binutils compile 
  make[4] -C toolchain/binutils install 
  make[4] -C toolchain/gcc prepare 
  make[4] -C toolchain/gcc compile 
  make[4] -C toolchain/uClibc compile 
  make[4] -C toolchain/uClibc install 
  make[4] -C toolchain/gcc install 
  make[2] -C /home/eric/kamikaze_7.09 target/compile 
  make[3] -C target compile 
  make[4] -C target/linux compile 
  make[5] -C target/linux/brcm-2.4 compile 
  make[4] -C target/linux/brcm-2.4/image compile 
  make[2] -C /home/eric/kamikaze_7.09 package/compile 
  make[3] -C package compile 
  make[4] -C package/base-files compile 
  make[4] -C package/bridge compile 
  make[4] -C package/busybox compile 
  make[4] -C package/dnsmasq compile 
  make[4] -C package/dropbear compile 
  make[4] -C package/iptables compile 
  make[4] -C package/nvram compile 
  make[4] -C package/broadcom-wl compile 
  make[4] -C package/broadcom-diag compile 
  make[4] -C package/zd1211_firmware compile 
  make[4] -C package/kernel compile 
  make[4] -C package/switch compile 
  make[4] -C package/wireless-tools compile 
  make[4] -C package/wlcompat compile 
  make[4] -C package/mtd compile 
  make[4] -C package/libpcap compile 
  make[4] -C package/linux-atm compile 
  make[4] -C package/ppp compile 
  make[2] -C /home/eric/kamikaze_7.09 package/install 
  make[3] -C package install 
  make[4] -C package install-targets 
  make[5] -C package/base-files install 
  make[5] -C package/bridge install 
  make[5] -C package/busybox install 
  make[5] -C package/dnsmasq install 
  make[5] -C package/dropbear install 
  make[5] -C package/iptables install 
  make[5] -C package/broadcom-wl install 
  make[5] -C package/broadcom-diag install 
  make[5] -C package/kernel install 
  make[5] -C package/switch install 
  make[5] -C package/wlcompat install 
  make[5] -C package/mtd install 
  make[5] -C package/nvram install 
  make[5] -C package/ppp install 
  make[5] -C package/wireless-tools install 
  make[4] -C package preconfig 
  make[2] -C /home/eric/kamikaze_7.09 target/install 
  make[3] -C target install 
  make[4] -C target/linux install 
  make[5] -C target/linux/brcm-2.4 install 
  make[4] -C target/linux/brcm-2.4/image compile 
  make[4] -C target/linux/brcm-2.4/image install 
  make[5] -C target/linux/brcm-2.4/image/lzma-loader clean install 
  make[4] -C target/sdk install 
  make[2] -C /home/eric/kamikaze_7.09 package/index 
  make[3] -C package index 
mockingbird@linuxbox:~/kamikaze_7.09$ 
```

That's all there is to compiling the SDK The SDK tar.bz2 file is now in the bin subdirectory. You can install it just as I describe above for the White Russian SDK that can be downloaded in its compiled form.

However, if you are working with Kamikaze 7.09 (the latest as of this writing) or before, you're likely to have trouble compiling packages with your newly compiled SDK. There's a bug in the Makefile in the root build directory. A fix has been added to the svn repository, but is not in any release yet. A description of the problem, (now patched) can be found [here](https://dev.openwrt.org/ticket/2508). To quickly fix this defect, extract the SDK directory and then edit the Makefile in the extracted root SDK directory. Open the file with your favorite editor and replace this code:

Excerpt from ~/kamikaze-sdk/Makefile (Broken):

```Makefile
world: FORCE
	$(MAKE) package/compile
	-( \
		cd package; \
		find . -maxdepth 2 -name Config.in | \
			sed -e 's,/Config.in,,g' | \
			xargs -n1 $(MAKE) compile -C; \
	)
```

With this, fixed, code: 

Excerpt from ~/kamikaze-sdk/Makefile (Fixed):

```
world: FORCE
	$(MAKE) package/compile
	-( \
		cd package; \
		for configfile in `find . -maxdepth 2 -name Config.in` ; do \
			$(MAKE) compile -C `dirname $$configfile` ; \
		done \
	)
```

You should now be able to compile your code just as you did with the White Russian SDK. 

I toyed with the idea of not reiterating, for a third time, that I only wish to be contacted if you find a problem with this tutorial, but given the number of people who have ignored my first two pleas, I decided better of it. I would also like to say that if you do need to contact me, it would be best to compose your message in English as opposed to l33t speak. I do not in any way mean to disparage those of you for whom English is not your first language by this statement, but merely those who have sent me messages containing more numeric and puctuation characters than alphabetic ones. Finally, some of you have also seen fit to ask for my help in exchange for money. Admittedly, I am not entirely opposed to this idea. However, if you really are interested in hiring me, please offer me a figure with at least three digits in it. Thank you. 

This tutorial is provided under the [Creative Commons License, version 3.0](http://creativecommons.org/licenses/by/3.0/us/).

### References

* <http://www.gargoyle-router.com/wiki/doku.php?id=openwrt_coding#part_iiibuilding_and_using_the_kamikaze_sdk_--_12_9_2007>

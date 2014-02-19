

Table of Hardware
=================

A very reliable way to check for existing support is to take a look at <http://downloads.openwrt.org/snapshots/trunk/>. The devices are sorted by target rather than manufacturer and if there is an image for the device, it should work (Bleeding Edge does contain bugs). Note that with the release of 'Attitude Adjustment (12.09 final)' on 25th April 2013, "Lower end devices with only 16 MiB RAM will easily run out of Memory…". Recommended image for bcm47xx based devices is Backfire with brcm-2.4.

If you want to add a device to the ToH, please distinguish between supported, WiP(Work in Progress) and so on. Then use the [template_device](http://wiki.openwrt.org/meta/template_device) to create a new page for that device. Also, this wiki is supervised but only little groomed, so do not just dump your half-digested stuff here and expect others to finish your work. It'll probably never happen. We try to countermeasure this by modularizing the articles as much as possible, so use the template. It contains link to articles you probably don't know of.

### Legend

* The **VLAN** column states whether the Ethernet device is capable of hardware VLANs, means whether the switch can be reprogrammed to generate tagged frames on certain ports, supports trunking etc. Tagged software VLANs (through vconfig) are possible in any case.
* The Status column contains the first OpenWrt version that fully supported the given model, unless otherwise stated it should apply to all subsequent releases. In doubt, consult the model specific article.
* About Platform you find some info here: <https://dev.openwrt.org/wiki/platforms>
* The Target column specifies the name of the OpenWrt port ("architecture") used on the device.
* See [buyer's gui](http://wiki.openwrt.org/toh/buyerguide) for some explanations of the features.
* This page does not contain information on simultaneous dual band-capability, you need to get this information from somewhere else.
* For finding your device quicker, see [tags](http://wiki.openwrt.org/meta/tags).

### Supported Hardware - Router type

Devices listed in this table have full or reasonably complete support and can be considered ready for use.

#### Evaluation boards / unbranded boards


Model | Version | Status | Target(s) | Platform | CPU Speed (MHz) | Flash(MB) | RAM(MB) | Wireless NIC | Wireless Standard | Wired Ports | VLAN Config | USB | SDIO
------|---------|--------|-----------|----------|-----------------|-----------|---------|--------------|-------------------|-------------|-------------|-----|-----
Ralink V11ST-FE | | R30561 | ramips | Ralink rt288x | | 4 | 32 | 1x Mini-PCI slot | | 4 * 100 | yes | No | No

#### 3Com

Model | Version | Status | Target(s) | Platform | CPU Speed (MHz) | Flash(MB) | RAM(MB) | Wireless NIC | Wireless Standard | Wired Ports | VLAN Config | USB | SDIO
------|---------|--------|-----------|----------|-----------------|-----------|---------|--------------|-------------------|-------------|-------------|-----|-----
3CRWER100-75 | No | trunk | atheros | Atheros 2315A | 180 | 4 | 16 | Atheros (integrated) | 11 b/g | 1+4 | ?? | No | No

#### 4G Systems

#### 8devices

#### Abicom International

#### Actiontec

#### Accton

#### ADB

#### Alcatel-Sbell

#### ALFA Network

#### Allnet

#### Alpha Networks

#### ARC Flex

#### Arcadyan / Astoria

#### AsiaRF

#### Asus

#### Atmel

#### Avm

#### Aztech

#### Belkin

#### Buffalo

#### CEEDTec

#### Catch Tec

#### Cobalt Networks

#### Compex

#### Comtrend

#### D-Link

#### Devolo

#### Dragino

#### Edimax

#### Engenius

#### Embedded Wireless

#### Fon

#### Freecom

#### Gateway

#### Gateworks

#### Gigaset

#### Huawei

#### jjPlus

#### Linksys

#### Meraki

#### Mercury

#### MikroTik

#### NetComm

#### Netgear

#### Olimex

#### PC Engines

#### Petatel

#### Pirelli

#### Planex

#### Poray

#### Qemu

#### Qi hardware

#### RaidSonic

#### Redwave

#### Rosewill

#### Sagem

#### Seagate

#### Sercom

#### SFR (Société Française de Radiotéléphonie)

#### Skyline

#### SimpleTech

#### Sitecom

#### SMC

#### Sparklan

#### Telsey

#### Tenda

#### Texas Instruments

#### TP-Link

#### Trendnet

#### T-Com / Telekom

#### Ubiquiti

#### Unbranded

#### Upvel

#### US Robotics

#### Vizio

#### Western Digital

#### Zcomax

#### ZTE

#### ZyXEL


### Supported Hardware - Devboard, Phones

Devices listed in this table have full or reasonably complete support and can be considered ready for use, otherwise this section contains only developer board, phones and other cool stuff that are not exactly "wireless router" out of the box.

#### Atmel AT91SAM

#### Freescale i.MX23/i.MX28

#### Freescale MPC85xx

### Work In Progress

Devices in this table have some level of support but are not considered ready for general use. One or more major aspects of the device may not be functioning. Please click on the model name to view the device page which should provide more details on the status of the progress.

#### 4ipnet

#### Actiontec

#### Agestar

#### Airlink101

#### Arcadyan / Astoria

#### Asmax

#### Asus

#### AudioCodes

#### Belkin

#### BT

#### Buffalo

#### Comtrend

#### D-Link

#### Davolink

#### Gigaset

#### Hame

#### Hi-Link

#### Huawei

#### Inventel

#### Iomega

#### Linksys

#### Logitech

#### Medion

#### Mercury

#### MikroTik

#### Netgear

#### Nprove

#### Openmoko

#### Panasonic

#### Patriot Memory

#### Pirelli

#### Poray

#### Raspberry Pi

#### Samsung

#### Seagate

#### Siemens

#### Sony

#### Thomson

#### TP-Link

#### Trendnet

#### US Robotics

#### VTech

#### Wiligear

#### Widemac

#### ZyXEL

### Possible but not being worked on

Devices in this table could have OpenWrt

#### ADB

#### Arcadyan / Astoria

#### Asus

#### Avm

#### Belkin

#### Buffalo

#### CC&C

#### Cisco

#### Colubris

#### D-Link

#### Dovado

#### Edimax

#### EnGenius

#### Fon

#### Hewlett-Packard (HP)

#### Huawei

#### Kingston

#### Kintec

#### Linksys

#### Motorola

#### Netcomm

#### Netgear

#### Netopia

#### Prolink

#### Rollei

#### QNAP

#### RCA

#### Sagem

#### Sapido

#### Scientific Atlanta

#### Sercom

#### Siemens

#### Sitecom

#### Sweex

#### Telsey

#### Teltonika

#### Tenda

#### Thomson

#### TP-Link

#### Trendnet

#### Xavi

#### ZLMnet

#### Zyxel

#### ZTE

### Unknown

Devices here are unknown, info is rough. Once info is complete, trusted, entries can move elsewhere.

#### TrendNet
ZLMNet

### Unsupportable

Devices in this table will not run OpenWrt. Common reasons include:

* insufficient quantity of flash (less then 4?MiB) or RAM (less then 16?MiB)
* device is based on a platform with no Linux support (unless somebody is willing to write drivers from scratch)
* device is too old (not obtainable any more, unimpressive technological features) to be worth the effort

#### 3Com

#### Airlink101

#### Asus

#### Canon

#### D-Link

#### Edimax

#### LG

#### Linksys

#### Netgear

#### Sagem

#### Sempre

#### Sitecom

#### SMC

#### Thomson

#### TP-Link

#### Trendnet

#### US Robotics


---

via: <http://wiki.openwrt.org/toh/start>

本文由 [AKmaker](https://github.com/AKmaker/openwrt-cn) 原创翻译，[AKmaker](http://akmaker.com) 荣誉推出

译者：[译者ID](https://github.com/译者ID) 校对：[校对者ID](https://github.com/校对者ID)


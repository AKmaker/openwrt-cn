
### Serial Cables
This article is aimed at what hardware is needed to communicate serially with a router or similar OpenWrt device. while this page is under construction feel free to add your device into **the A-Z list**. Then add a description of how it works, links and so on if required.

### A-Z Summary

Name & Description                 | Host OS support | Quality  | Output Voltage | Host Connection
-----------------------------------|-----------------|----------|----------------|----------------
Arduino Development board          | Compatible      | Reliable | 0v and 5V      | USB
Data cable for Siemens cell phones | Compatible      | Reliable | 0v and 5V?     | USB

### Arduino Development Boards

1. Remove the programmable IC
2. Connect the RX(0) and TX(1) and the development board ground through to the router/device.
3. Use software to talk to the router/device.

This method actually uses the Development boards built in USB to RS232 converter and results in 5V signals.

Note: compatible development boards include: Demulotev.

### Arduino Uno(REV3) as USB to serial cable

Caution: Do this at your own risk, since the Arduino runs at 5v and the serial sometimes is at a different voltage. Therefore, using the following technique may cause the router, Arduino, or other things, to malfunction or break. That said, this technique worked well for me during for lots of use (hours), with a [Linksys E1000 v1](http://wiki.openwrt.org/toh/linksys/e1000) at 3.3v, and an Arduino Uno REV3.

Needed:

1. An Arduino Uno REV3 (may work on other arduinos, but only tested on the above)
2. A router
3. A computer
4. A USB A-B cable (to use with the Arduino)
5. Some wire and soldering tools

By the way this method does not require any hardware modifications to the arduino.

Upload [the Arduino Bare Minimum sample sketch](http://arduino.cc/en/Tutorial/BareMinimum) (consisting of empty setup() and loop() functions) to the Arduino. Then solder one end of the wires to the serial port (some devices may need the Vcc to be connected, and some may not). Then connect the ground to the arduinos ground, and the TX and RX pins (digital 0 and 1 – they're labeled as TX and RX) of the arduino to the TX and RX of the device (maybe the other way around, I don't remember; try switching them if it's not working, as far as I know it shouldn't break if reversed.). Then connect the Arduino to a computer and connect. (The Arduino utility is not needed; just use a regular serial terminal program).

As far as I know, this technique works since the Arduino Uno REV3 has a chip called the atmega16U2, which is a serial to USB converter, to talk to the computer; we use it here to talk to the router. So, other arduinos may work.

### Data cable for Siemens cell phones

This kind of cable is really cheap. Only some €uros on ebay, because the phones are not built any more.

The cable should be compatible to one of these Siemens cell phones:
A35 / A36 / A50 / C25 / C25 Power / C28 / C35i / C45 / M35i / M50 / ME45 / MT50 / S25 / S35i / S45 / S45i / SL42 / SL45 / SL45i

It's the well supported prolific pl2303 USB to serial converter. There are 4 lines coming from the usb part to the phone jack. You don't need the Vcc line.

pin | function
----|---------
1   | GND
3   | Vcc
5   | Tx
6   | Rx

[Mesh cube wiki page from archive.org](http://web.archive.org/web/20070820095200/http://www.meshcube.org/meshwiki/ModifiedMobileSerCable)

### PL2303 cables

It's a component of many cheap USB-TTL cables available on ebay. Prolific produces the PL-2303HX/PL-2303HX.D chip in different variants at least since 2002 ( [PL2303HX datasheet](http://www.stkaiser.de/anleitung/files/PL2303.pdf), [PL2303HXD datasheet](http://www.prolific.com.tw/UserFiles/files/ds_pl2303HXD_v1_4_4.pdf) ). There are different designt with unattached cables, 3 pin (no 3.3V / 5V ), 4 pin (likely 3.3V ) or more pins available. Connecting 3.3V/5V is not needed and can damage the board.

### name?

<https://www.adafruit.com/products/70> is a great serial cable. It's used for ttl level serial. (If you have to solder to the board, it's probably ttl. If there is an actual serial port, then it's 12 v serial. Different device)

### Refs

* <http://wiki.openwrt.org/doc/hardware/port.serial.cables>

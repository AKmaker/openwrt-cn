
### Pre

* [GPIO](http://en.wikipedia.org/wiki/General%20Purpose%20Input/Output)
* [GPIO access conventions on Linux](http://www.mjmwired.net/kernel/Documentation/gpio.txt)
* [mmc_over_gpio](http://wiki.openwrt.org/doc/howto/mmc_over_gpio)
* [GPIOs of AR913x SoC](http://wiki.openwrt.org/toh/tp-link/tl-wr1043nd#gpios)
* [oldwiki GPIO](http://wiki.openwrt.org/oldwiki/port.gpio)

### Hardware

GPIOs are commonly used in router devices for buttons or leds. They only safely supply or sink (pull to GND) a maximum of 4mA aprox., and the voltage is usually 3V when active. Only two states are posible: high or low. Depending on how a device is activated by a GPIO, active low or active high is defined.

* **active high**: the device is activated when the GPIO is HIGH.

* **active low**: the device is activated when the GPIO is LOW.

In this image you can see how a GPIO is wired to buttons or leds, to work as active low or high.

GPIOs can be also used for complex tasks: 
       |    kernel module   | description
-------|--------------------|-------------
1-wire | kmod-w1-master-gpio| 1-wire bus master
PWN    | kmod-pwm-gpio      | pulse width modulator
SPI    | kmod-spi-gpio      | bitbanging Serial Peripheral Interface
       | kmod-mmc-over-gpio | [MMC/SD card over GPIO](http://wiki.openwrt.org/doc/howto/mmc_over_gpio)
 I2C   | kmod-i2c-gpio      | bitbanging I2C
LIRC   | no module yet      | Linux Infrared Remote Control
Rotary encoder | kmod-input-gpio-encoder | GPIO rotay encoder

### GPIO Interrupts

GPIO interrupts are useful when a GPIO is used as input and you need to manage high signal frequencies. Without interrupts, GPIO inputs must be managed using the **polling** method. With polling you cannot manage signal inputs with high frequencies. 

Not all boards have GPIO interrupts. For example **bcm63xx SoCs haven't GPIO interrupts** and that's because their buttons are polled. As a result of this, some input drivers listed above won't work in these boards. 

### Software

In linux GPIOs can be accesed through GPIO SYSFS interface: **/sys/class/gpio/**

To control GPIOs you can use **gpioctl-sysfs**. Also manually or with this simple script you can control GPIOs not used by buttons or leds. 

```bash
#!bin/sh
 
show_usage()
{
    printf "\ngpio.sh <gpio pin number> [in|out [<value>]]\n"
}
 
if [ \( $# -eq 0 \) -o \( $# -gt 3 \) ] ; then
    show_usage
    printf "\n\nERROR: incorrect number of parameters\n"
    exit 255
fi
 
#doesn't hurt to export a gpio more than once
(echo $1 > /sys/class/gpio/export) >& /dev/null
 
if [  $# -eq 1 ] ; then
   cat /sys/class/gpio/gpio$1/value
   exit 0
fi
 
if [ \( "$2" != "in" \) -a  \( "$2" != "out" \) ] ; then
    show_usage
    printf "\n\nERROR: second parameter must be 'in' or 'out'\n"
    exit 255
fi
 
echo $2 > /sys/class/gpio/gpio$1/direction
 
if [  $# -eq 2 ] ; then
   cat /sys/class/gpio/gpio$1/value
   exit 0
fi
 
 
VAL=$3
 
if [ $VAL -ne 0 ] ; then
    VAL=1
fi
 
echo $VAL > /sys/class/gpio/gpio$1/value
```

### Refs

* <http://wiki.openwrt.org/doc/hardware/port.gpio>

* <https://forum.openwrt.org/viewtopic.php?id=36561>

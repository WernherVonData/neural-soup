---
layout: post
title:  "BananaPI GPIO"
date:   2023-07-27 18:00:00 +0200
categories: bananapi update
---
## Introduction

So when the board is up and running and greets with such beautiful screen:

![armbian welcome](./img/armbian_welcome.png)

Then there is a urge to do something. What it can be? GPIO - let's check if it's working. Had to figure it out since the physical pins have different numbers when they are mapped in the system.

## Know your pin

Apparently information about the pin number is not that hard when you know where to look.

First - see the pin identifier - [wiki](https://wiki.banana-pi.org/Banana_Pi_BPI-M2_Berry#GPIO_PIN_define) is helping you with that.

For example let's say we want to want to put a led to pin number 7. We are connecting the `+` to pin 7 and `-` to 6 (ground). Keep in mind to add some resistor to avoid burning your led.

Second - check the mapping in your kernel - it took me a while to figure it out, but the wiki for kernel was helpful. Kernel in my BPI is sunxi, so its [wiki](https://linux-sunxi.org/GPIO) told me everything I wanted to know.

The physical pin 7 on the BPI is called `PB3`. Going with the formula from sunxi wiki:
```
(position of letter in alphabet - 1) * 32 + pin number
```
I have:
```
(B-1)*32 + 3 = (2-1)*32+3 = 35
```

It's easy to check, just
```
cat /sys/kernel/debug/pinctrl/*/pinmux-pins // sudo is most likely required
```
grep for `PB3` and you will see the result.

## The C way to control your leds - gpiod

There are plenty of ways to control the gpio: 
* You can use [WiringPI](http://wiringpi.com/) (at least some people said it was working on their BPIs, but for me it doesn't) - the library is developed directly for RaspberryPI and since there are differences in the hardware - it may not work correctly on BananaPI.
* Use sysfs to control the pins (tested, worked), but ["is now deprecated in favor of character devices /dev/gpiochipX"](https://linux-sunxi.org/GPIO#Accessing_the_GPIO_pins_through_character_device_with_mainline_kernel)
* Use C and [libgiod](https://git.kernel.org/pub/scm/libs/libgpiod/libgpiod.git/) library.

Used the library on my own you can see the code [here](https://github.com/WernherVonData/bananapi-things/tree/main/led-by-gpio) - turns on/off the pin `PB3` five times and then it's going off.

For direct use of gpiod I recommend the [wiki](https://linux-sunxi.org/GPIO#Accessing_the_GPIO_pins_through_character_device_with_mainline_kernel) - do not see a reason to repeat the same stuff here.

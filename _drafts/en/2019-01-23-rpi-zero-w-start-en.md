---
layout: post
title: "[RaspberryPi zero W] headless setup"
categories:
  - Raspberry Pi
date: 2019-01-23
tags: [raspberrypi, rpizero, headless, wlan]
---

I have a chance to try Raspberry Pi zero.

There are two types of RPI zero models depending on whether or not they support wireless LAN.
In this article, I will try to organize the failure experience by proceeding with initialization of RPI zero.

The product looks like this. (The extension headers had to be soldered separately.)

![Raspberry Pi Zero W](/files/rpizerow_board.jpg){: width="70%"}

## 환경 준비

### Material

- Micro sd card: 최소 4GB 이상
- SD card reader

### Preparation

- Windows 10 64bit

* Raspbian image

Raspbian uses the following versions because it will be used to compare two versions of 4.9.x & 4.14.x.

- 2017-11-29-raspbian-stretch-lite
- 2018-03-13-raspbian-stretch-lite

- Raspbian version information

  - [Rasbian release note](http://downloads.raspberrypi.org/raspbian/release_notes.txt){:target="\_blank"}

- Download previous version of Raspbian
  - [Raspbian image download](http://downloads.raspberrypi.org/raspbian/images/){:target="\_blank"}
  - [Raspbian Lite image download](http://ftp.jaist.ac.jp/pub/raspberrypi/raspbian_lite/images/){:target="\_blank"}

### Headless

Headless setup refers to setting without display. When I do a word search, it usually means that there is no monitor.

In case of Raspberry pi 3, there is enough interface such as USB, so you can set it directly by connecting monitor, keyboard and mouse, but it is very cumbersome to connect it.

Especially since the RPI zero model has only one USB port for micro 5pin, it is difficult to set up through a monitor unless there is a hub that can expand the port.

**In many ways, headless setup is much simpler.**

---

## Raspberry Pi zero W headless setup

The Raspberry Pi zero W is compact and does not have an Ethernet connector, instead it supports WiFi.

So when I did a search, there were several ways to set headless for RPI zero W products.

I made the following configuration with search results.

---

### Installing raspbian on Micro SD card (image write)

Refer to the below article for details on how to install raspbian on Micro SD card. (up to ssh enable configuration)

[[RaspberryPi3] 라즈베리파이3 - 라즈비안(Raspbian) OS 설치](https://inmile.tistory.com/6?category=793890){:target="\_blank"}

---

### Create wpa_supplicant.conf to configure the wireless LAN - Failed Note

Create an empty file named wpa_supplicant.conf and write the following:

_**Configuration files that did not work**_

```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
country=US
update_config=1

network={
  ssid="your_ssid"
  psk="your_pw"
  key_mgmt=WPA-PSK
}
```

This file is also put in the same location as the empty ssh file (boot partition).

And insert the sd card into RPI zero W and connect the power.
It takes about 90 ~ 120 seconds to set the initial time, and then try to connect ssh to raspberrypi.local.

But...

**I can not connect.**

I thought that something's wrong.
I uploaded a new image, tried to change the contents of the configuration file and tried everything but it still is not.

It's not a complicated configuration, but not working...
I tried the search again.

Once I have given up the haedless configuration through the wlan and soldered the extension header to check the cause, connect the serial cable to the RPI zero as shown in the following picture.

To use serial, you need to add the following option to the boot config.txt file.

```
# Enable uart
enable_uart=1
```

![Raspberry Pi Zero W - serial connection](/files/rpizerow_serial.jpg){: width="70%"}

After connecting with serial, I tried wifi configuration with raspi-config, and the following message printed. There seemed to be a problem with the WLAN configuration.

```
Could not communicate with wpa_supplicant.
There was an error running option N2 Wi-fi.
```

---

### Create wpa_supplicant.conf to configure the wireless LAN - Success!

While I was checking and searching through the serial for the problem on the wireless LAN side, I found the wpa_supplicant debugging information.

The following link shows the wpa_supplicant command.

[Raspberrypi forum: Raspbian Stretch: Wifi not starting on boot](https://www.raspberrypi.org/forums/viewtopic.php?t=191061&start=25){:target="\_blank"}

When I connected to RPI zero in serial and executed the following command, an error was output.

```
\$ sudo wpa_supplicant -d -B -c/etc/wpa_supplicant/wpa_supplicant.conf -iwlan0
wpa_supplicant v2.4
random: Trying to read entropy from /dev/random
Successfully initialized wpa_supplicant
Initializing interface 'wlan0' conf '/etc/wpa_supplicant/wpa_supplicant.conf' driver 'default' ctrl_interface 'N/A' bridge 'N/A'
Configuration file '/etc/wpa_supplicant/wpa_supplicant.conf' -> '/etc/wpa_supplicant/wpa_supplicant.conf'
Reading configuration file '/etc/wpa_supplicant/wpa_supplicant.conf'
ctrl_interface='DIR=/var/run/wpa_supplicant GROUP=netdev'
country='US'
update_config=1
Line 4: unknown global field ' '.
Line 4: Invalid configuration line ' '.
Line 6: unknown network field '  ssid'.
Line 7: unknown network field '  psk'.
Line 9: failed to parse network block.
Failed to read or parse configuration '/etc/wpa_supplicant/wpa_supplicant.conf'.
Failed to add interface wlan0
: Cancelling scan request
: Cancelling authentication timeout
```

The first problem was that I did not recognize the space in the network configuration section to set the ssid and password.

Modify spaces to tabs and perform the command again.

```
wpa_supplicant v2.4
random: Trying to read entropy from /dev/random
Successfully initialized wpa_supplicant
Initializing interface 'wlan0' conf '/etc/wpa_supplicant/wpa_supplicant.conf' driver 'default' ctrl_interface 'N/A' bridge 'N/A'
Configuration file '/etc/wpa_supplicant/wpa_supplicant.conf' -> '/etc/wpa_supplicant/wpa_supplicant.conf'
Reading configuration file '/etc/wpa_supplicant/wpa_supplicant.conf'
ctrl_interface='DIR=/var/run/wpa_supplicant'
Line 2: unknown global field 'GROUP=netdev'.
Line 2: Invalid configuration line 'GROUP=netdev'.
...
```

The second problem was that it did not recognize the 'GROUP = netdev' option.

The posting date I refer to in the wpa_supplicant.conf file was early in the year of 2017, did the Raspbian version go up and change something?

This part seems to be checked.

Anyway, it works well to remove the option and turn it back on.

The final configuration file created by debugging is as follows.

#### wpa_supplicant.conf

(The network part should be written in tab.)

```
ctrl_interface=DIR=/var/run/wpa_supplicant
country=US
update_config=1
network={
    ssid="your_ssid"
    psk="your_pw"
    key_mgmt=WPA-PSK
}
```

Applying the above configuration file works well.

## Summary

The process is long, but in summary:

- Prepare a micro sd card and write raspbian image.
- Add an empty ssh file to the boot drive and create wpa_supplicant.conf for WiFi configuration.
- Insert sd card into RPI zero and wait 1~2 minutes after connecting power.
- connect ssh to raspberrypi.local (Account: pi / raspberry)
  - ping before connecting. (\$ ping raspberrypi.local)

If any other raspberry pi is connected, disconnect it.

This is the end of the initial setup.

In the next article, I will summarize how to add Ethernet using WIZ850IO module.

---

---
layout: post
title: "[RaspberryPi zero W] Add ethernet to Raspberry Pi zero W with WIZ850io (1)"
categories: RaspberryPi
date: 2019-01-29 00:00
tags: [raspberrypi, rpizero, wiz850io, w5500]
---

In this article, I will summarize how to add Ethernet by connecting IO module to Raspberry Pi zero W (RPI zero).

The product is a module called WIZnet's WIZ850io, which is based on the W5500 Ethernet chip and includes an RJ45 connector.

<img src="https://wizwiki.net/wiki/lib/exe/fetch.php?media=products:wiz850io:wiz850io.png" width="30%" />

The W5500 is an Ethernet controller chip with hardwired TCP / IP and supports SPI communication.
It helps to easily add internet connection to embedded system.

Details of the WIZ850io module can be found on the link below.

- [WIZwiki - WIZ850io](https://wizwiki.net/wiki/doku.php?id=products:wiz850io:start)
- [WIZ850io shop link](http://shop.wiznet.co.kr/front/contents/product/view.asp?cateid=48&pid=1263){:target="\_blank"}

I will add Ethernet to the RPI zero by connecting the WIZ850io module through the SPI interface.
By adding Ethernet, you can use the Internet more reliably than WiFi.

The figure below shows the result of pinging the connected router for each network (WiFi / Ethernet).

| When using only WiFi (wlan0)   | When using only wired LAN (ethernet: eth0) |
| ------------------------------ | ------------------------------------------ |
| ![](/files/wlan_vs_eth-01.png) | ![](/files/wlan_vs_eth-02.png)             |

This can vary depending on the environment, but I think it is enough to confirm the difference stability.

---

The process can be divided into the following.

In this article, we will cover RPI zero initialization and kernel compile.

- RPI zero 초기 설정
  - 이전 글 참조: [[RaspberryPi zero W] headless 초기 설정]({{ site.baseurl }}{% post_url 2019-01-23-rpi-zero-w-start %})
- RPI zero kernel compile
- Device tree 작성 & 적용 (for W5500)
- 동작 테스트

---

### Development environment

The development environment is as follows. Raspbian used the 2018-03-13 version, not the latest version, for kernel compatibility testing.

- Ubuntu 16.04 64bit VM (for cross-compile)

- Raspbian version
  - 2018-03-13-raspbian-stretch-lite (Linux kernel 4.9.80+)

### Pin connection

Below is the pin connection information of RPI zero and WIZ850io.

Each pinmap can be found at the following link. For RPI zero, the direction may be confusing, from the side with the micro sd card slot to the one.

- [WIZ850io specification](https://wizwiki.net/wiki/doku.php?id=products:wiz850io:start){:target="\_blank"}
- [Raspberry Pi pinout](https://pinout.xyz/){:target="\_blank"}

| RPI zero       | WIZ850io |
| -------------- | -------- |
| 6 (GND)        | GND      |
| 1 (3.3V)       | 3.3V     |
| 23 (SPI0_SCLK) | SCLK     |
| 19 (SPI0_MOSI) | MOSI     |
| 21 (SPI0_MISO) | MISO     |
| 18 (BCM24)     | RSTn     |
| 15 (BCM22)     | INTn     |
| 24 (SPI0_CE0)  | SCNn     |

When all the pins are connected, it looks like the picture below.

![Raspberry Pi Zero W - WIZ850io](/files/rpi-zero-with-wiz850io_1.jpg){: width="50%"}

---

### Raspberry pi kernel compile

How to compile the kernel for Raspberry pi is explained in the link below.

- [Raspberry Pi kernel build](https://www.raspberrypi.org/documentation/linux/kernel/building.md){:target="\_blank"}

This article briefly summarizes the cross compile process for RPI zero.

In the ubuntu environment in which the cross compile is performed, proceed as follows.

#### Required package installation and toolchain setup

```
$ sudo apt-get install git bc
$ sudo apt-get install libncurses5-dev libncursesw5-dev
# Toolchain 설치
$ git clone https://github.com/raspberrypi/tools ~/tools
$ echo PATH=\$PATH:~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin >> ~/.bashrc
$ source ~/.bashrc
```

#### Kernel source download

I set up the branch name to download to match the version of the Raspbian image to use and the kernel version.

The kernel information for each Raspbian version can be found at the following link:

- [Rasbian release note](http://downloads.raspberrypi.org/raspbian/release_notes.txt){:target="\_blank"}

```
# specific version download
git clone --depth=1 --branch rpi-4.9.y https://github.com/raspberrypi/linux

# latest version download
git clone --depth=1 https://github.com/raspberrypi/linux
```

#### Kernel configuration (for RPI zero)

Once the download is complete, proceed to the kernel configuration and menuconfig settings for the W5500 driver.

```
$ cd linux
$ KERNEL=kernel
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcmrpi_defconfig

# W5500 driver config
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

![menuconfig 화면](/files/rpi-zero-menuconfig_1.png)

![menuconfig 검색 창](/files/rpi-zero-menuconfig_2.png)

Type '/' to display the search box, and search for the 'wiznet' keyword.

In the search results, you can move the page by pressing the direction key or page up / down button.

![W5500 설정 전](/files/rpi-zero-menuconfig_3.png)

Activate two options here.

Select option #1: 'WIZnet W5100 Ethernet support'

If you enable the above option, the following options will appear.

Select option #2: 'WIZnet W5100 / W5200 / W5500 Ethernet support for SPI mode'

You can also select a new option.

![W5500 설정 후](/files/rpi-zero-menuconfig_4.png)

![마지막에 저장하겠냐는 팝업이 나타난다.](/files/rpi-zero-menuconfig_5.png)

- Note: How to choose options
  - Use the up and down arrow keys to move to the menu to set and **press the space bar to select it**.
  - Legend: [*] built-in [ ] excluded <M> module < > module capable
  - When you have completed your selection, use the left and right arrow keys to select Exit from the menu at the bottom, then press enter to exit and, if the popup appears at the end, select Yes to save your settings.

#### Kernel compile

When the configuration is completed, proceed with compile.

```
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs -j4
```

It takes a long time to complete the compilation. It took about 30 minutes to an hour, depending on the PC specification.

The kernel images and modules we will use are created in the following path, respectively. Copy the following files to raspberry pi.

- Kernel image:
  - linux/arch/arm/boot/zImage
- modules:
  - linux/drivers/net/ethernet/wiznet/w5100.ko & w5100-spi.ko

There is also a way to copy directly using sd card, but since there are not many files here, we will use a file transfer tool called **scp**. At this time, RPI zero and PC communication should be possible.

Copy the file using the following command.

```
$ scp arch/arm/boot/zImage pi@<RPI zero IP address>:/home/pi
$ scp drivers/net/ethernet/wiznet/*ko pi@<RPI zero IP address>:/home/pi
```

After completing the article up to this point, I will go through the following article to write, apply, and test the device tree.

---
layout: posts
title: "[RaspberryPi zero W] 이더넷 추가 with WIZ850IO (1)"
categories: RaspberryPi
date: 2019-01-29
tags:
  - raspberrypi
  - 라즈베리파이
  - wiz850io
  - w5500
---
<!-- Add ethernet to Raspberry Pi zero W with WIZ850IO -->

이번 글에서는 Raspberry Pi zero W (이하 RPI zero)에 IO 모듈을 연결하여 이더넷을 추가하는 방법을 정리해 본다.

사용 제품은 WIZnet의 WIZ850IO 라는 모듈로, 이더넷 칩인 W5500과 RJ45 커넥터를 포함하고 있다.

<img src="https://wizwiki.net/wiki/lib/exe/fetch.php?media=products:wiz850io:wiz850io.png" width="30%" />

W5500은 hardwired TCP/IP가 내장된 이더넷 컨트롤러 칩으로, SPI 통신을 지원한다. 임베디드 시스템 등에 쉽게 인터넷 연결을 추가할 수 있도록 도와준다.

RPI zero에 SPI 인터페이스를 통해 WIZ850IO 모듈을 연결하여 이더넷을 추가할 것이다. 이더넷을 추가하면, 무선랜(WiFi)보다 안정적으로 인터넷을 사용할 수 있다.

WIZ850IO 모듈의 상세 정보는 아래 링크에서 확인할 수 있다.

- [WIZwiki - WIZ850io](https://wizwiki.net/wiki/doku.php?id=products:wiz850io:start)
- [WIZ850IO 구매처](http://shop.wiznet.co.kr/front/contents/product/view.asp?cateid=48&pid=1263){:target="\_blank"}

---

과정은 크게 다음과 같은 단계로 나눌 수 있을 것 같다.

- RPI zero 초기 설정
  - 이전 글 참조: [[RaspberryPi zero W] headless 초기 설정](https://renakim.github.io/2019/01/23/rpi-zero-w-start/)
- RPI zero kernel compile
- Device tree 작성 & 적용 (for W5500)
- 동작 테스트

### 개발 환경

개발 환경은 다음과 같다. 
Raspbian은 커널 호환 테스트 문제로 최신 버전이 아닌 2018-03-13 버전을 사용했다.

- Ubuntu 16.04 64bit VM (for cross-compile)

- Raspbian version
  - 2018-03-13-raspbian-stretch-lite (Linux kernel 4.9.80+)

### Pin 연결

아래는 RPI zero와 WIZ850IO의 pin 연결 정보이다.

각각의 Pinmap은 다음 링크에서 확인할 수 있다.
RPI zero의 경우 방향이 헷갈릴 수 있는데, micro sd card 슬롯이 있는 쪽부터 1번이라고 보면 된다.

- [WIZ850IO specification](https://wizwiki.net/wiki/doku.php?id=products:wiz850io:start){:target="\_blank"}
- [Raspberry Pi pinout](https://pinout.xyz/){:target="\_blank"}


| RPI zero       | WIZ850IO |
| -------------- | -------- |
| 6 (GND)        | GND      |
| 1 (3.3V)       | 3.3V     |
| 23 (SPI0_SCLK) | SCLK     |
| 19 (SPI0_MOSI) | MOSI     |
| 21 (SPI0_MISO) | MISO     |
| 18 (BCM24)     | RSTn     |
| 15 (BCM22)     | INTn     |
| 24 (SPI0_CE0)  | SCNn     |


Pin을 모두 연결하면 이런 모습이 된다.

![Raspberry Pi Zero W - WIZ850IO](/files/rpi-zero-with-wiz850io_1.jpg){: width="50%"}

----

### Raspberry pi kernel compile

Raspberry pi를 위한 커널 컴파일 방법은 아래 링크에 잘 설명되어 있다.

- [Raspberry Pi kernel build](https://www.raspberrypi.org/documentation/linux/kernel/building.md){:target="\_blank"}

여기에서는 RPI zero를 위한 cross compile 방법을 간략히 정리한다.

Cross compile을 진행할 ubuntu 환경에서 다음 과정을 진행한다.

#### 필수 패키지 설치 및 toolchain 설정

```
$ sudo apt-get install git bc
$ sudo apt-get install libncurses5-dev libncursesw5-dev
# Toolchain 설치
$ git clone https://github.com/raspberrypi/tools ~/tools
$ echo PATH=\$PATH:~/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin >> ~/.bashrc
$ source ~/.bashrc
```

#### Kernel source download

사용할 Raspbian 이미지와 커널 버전을 맞추기 위해 다운받을 브랜치 버전을 설정해 주었다.

각 Raspbian 버전의 커널 정보는 다음 링크에서 볼 수 있다.

- [Rasbian release note](http://downloads.raspberrypi.org/raspbian/release_notes.txt){:target="\_blank"}

```
# 특정 버전 download
git clone --depth=1 --branch rpi-4.9.y https://github.com/raspberrypi/linux

# 최신 버전 download
git clone --depth=1 https://github.com/raspberrypi/linux
```

#### Kernel configuration (for RPI zero)

커널 configuration 및 W5500 드라이버를 위한 menuconfig 설정을 진행한다.

![menuconfig 화면](/files/rpi-zero-menuconfig_1.png)

```
$ cd linux
$ KERNEL=kernel
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- bcmrpi_defconfig

# W5500 driver 설정
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- menuconfig
```

![menuconfig 검색 창](/files/rpi-zero-menuconfig_2.png)

'/'를 입력하여 검색 창을 띄우고, wiznet 키워드로 검색한다.

검색 결과에서 방향키나 page up/down 버튼을 누르면 페이지를 이동할 수 있고, 번호를 입력하면 해당 옵션으로 이동된다.

![W5500 설정 전](/files/rpi-zero-menuconfig_3.png)

여기에서 두 옵션을 활성화한다.

**'WIZnet W5100 Ethernet support'** 선택

위 옵션을 활성화하면 아래 옵션이 추가로 나타난다.

**'WIZnet W5100/W5200/W5500 Ethernet support for SPI mode'** 선택

새로 나타난 옵션도 선택해준다.

![W5500 설정 후](/files/rpi-zero-menuconfig_4.png)

![마지막에 저장하겠냐는 팝업이 나타난다.](/files/rpi-zero-menuconfig_5.png)

- 참고: 옵션 선택 방법
  - 상하 방향키를 이용해 설정할 메뉴로 이동한 후 스페이스 바를 눌러 선택한다.
  - 한번 누르면 <M> (module), 두번 누르면 <\*> (built-in)이 되며, 여기에서는 모듈 형태로 컴파일한다.
  - 선택을 완료했으면 좌우 방향키를 이용해 하단의 메뉴 중 Exit를 선택 후 enter 키를 눌러 빠져나가며, 마지막에 팝업이 나타나면 Yes를 선택해 설정을 저장한다.

#### Kernel compile

설정이 완료되었으면 compile을 진행한다.

```
$ make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- zImage modules dtbs -j4
```

컴파일이 완료되기까지는 시간이 꽤 걸린다. 30분~1시간 정도 걸렸던 것 같은데, 이는 PC 사양에 따라 달라질 수 있다.

우리가 사용할 커널 이미지와 모듈은 각각 다음 경로에 생성된다. 아래 파일들을 raspberry pi로 복사해야 한다.

- Kernel image: linux/arch/arm/boot/zImage
- modules: linux/drivers/net/ethernet/wiznet/w5100.ko & w5100-spi.ko

sd card를 이용해 직접 복사하는 방법도 있지만, 여기에서는 파일 개수가 많지 않기 때문에 scp라는 파일 전송 툴을 이용하기로 한다.
이 때 RPI zero와 PC 간 통신이 가능한 상태여야 한다.

아래 명령어를 사용해 파일을 복사하자.

```
$ scp arch/arm/boot/zImage pi@<RPI zero IP address>:/home/pi
$ scp drivers/net/ethernet/wiznet/*ko pi@<RPI zero IP address>:/home/pi
```

다음 글에서 Device tree 작성 및 적용, 테스트 까지 진행해 본다.

<!-- ### Device tree overlay

Device tree에 대한 설명은 다음 링크를 참조한다.

- [라즈베리 파이 문서: 10. 디바이스 트리, 오버레이, 파라미터](https://wikidocs.net/3205)
- [Raspberry Pi doc: Device Trees, overlays, and parameters](ttps://www.raspberrypi.org/documentation/configuration/device-tree.md)

아래 내용은 RPI zero에서 직접 진행해도 무방하다. -->

---
layout: posts
title: "[RaspberryPi zero W] 이더넷 추가 with WIZ850IO (2)"
categories: RaspberryPi
date: 2019-02-11
tags: [raspberrypi, rpizero, wiz850io, w5500]
---

지난 글 [[RaspberryPi zero W] 이더넷 추가 with WIZ850IO (1)]()에 이어서, Device tree를 작성하는 내용부터 진행해 본다.

- ~~RPI zero 초기 설정~~
  - ~~이전 글 참조: [[RaspberryPi zero W] headless 초기 설정](https://renakim.github.io/2019/01/23/rpi-zero-w-start/)~~
- ~~RPI zero kernel compile~~
- **Device tree 작성 & 적용 (for W5500)**
- **동작 테스트**

아래 내용은 RPI zero에 접속한 상태에서 직접 진행하도록 하자.

### Custom kernel 적용

지난 글 마지막 스텝에서 Host linux에서 컴파일 한 커널을 scp를 이용해 RPI zero로 복사해 주었었다.

RPI zero의 홈 디렉토리에 들어가 보면 zImage와 w5100.ko, w5100-spi.ko 파일이 있을 것이다.
아래 명령으로 확인하자.

```
$ cd
$ ls -al
```

커널 적용 방법은 두 가지로, 어느 방법으로 진행해도 무방하다.

#### 기존 커널 파일을 대체하는 방법

우선 복사된 커널 이미지를 kernel7.img로 rename 한다. (Raspberry pi boot에서 인식하는 커널 네임)

\$ mv zImage kernel7.img

그리고 기존 커널을 .org라는 이름으로 백업한 뒤 새 커널을 복사하고, reboot 한다.

```
$ sudo mv /boot/kernel7.img /boot/kernel7.img.org
$ sudo cp kernel7.img /boot/.
$ sudo reboot
```

부팅이 완료되면, 다음 명령어를 이용해 새 커널로 적용되었는지 확인해 본다.

```
$ uname -r
```

#### 기존 커널을 유지하고 config.txt에서 추가하는 방법

기존 커널을 그대로 두고 새로운 이름의 커널을 /boot/config.txt 에서 직접 지정해도 된다.

```
$ mv zImage kernel_w5500.img
```

새 이미지를 /boot/ 폴더로 복사 후, config.txt에서 파일명을 지정해 준다.

```
$ sudo vi /boot/config.txt
...
kernel=kernel_w5500.img

# 저장 후 reboot
$ sudo reboot
```

### Device tree overlay 작성

Device tree는 일정한 문법을 갖춘 텍스트를 이용하여 시스템의 hardware(SoC, Board)를 기술하는 것이다.

그리고 Device tree overlay는 SoC와 관련된 기본 device tree를 두고, 선택적 구성요소에 대한 device tree 조각을 추가하는 방법을 말한다.

Device tree에 대한 상세 설명은 아래 링크에서 확인할 수 있다.

- [라즈베리 파이 문서: 10. 디바이스 트리, 오버레이, 파라미터](https://wikidocs.net/3205)
- [Raspberry Pi doc: Device Trees, overlays, and parameters](ttps://www.raspberrypi.org/documentation/configuration/device-tree.md)

Device tree overlay는 사용 목적에 따라 여러 형태로 기술될 수 있다. 물론 문법은 지켜져야 한다.

다음은 W5500을 위한 device tree overlay 작성 예시이다.

```
$ vi w5500-overlay.dts

/dts-v1/;
/plugin/;

/ {
    compatible = "brcm,bcm2835";
    fragment@0 {
        target = <&spi0>;
        __overlay__ {
            #address-cells = <1>;
            #size-cells = <0>;

            status = "okay";

            spidev@0 {
                status = "disabled";
            };
            spidev@1 {
                status = "disabled";
            };
        };
    };

    fragment@1 {
        target = <&spi0>;
        __overlay__ {
            #address-cells = <1>;
            #size-cells = <0>;

            status = "okay";

            w5500@1 {
                compatible = "wiznet,w5500";
                reg = <0>;
                spi-max-frequency = <10000000>;
                mac-address = [00 08 DC 01 02 03];
                interrupts = <22 0x8>;
                interrupt-parent = <&gpio>;
                status = "okay";
            };
        };
    };
};
```

dts 파일 작성이 완료되었으면, dtc를 이용해 컴파일한 뒤 /boot/overlay 폴더에 복사한다.

```
# dts compile
$ dtc -I dts -O dtb -o w5500-overlay.dtb w5500-overlay.dts

# dtb 파일 복사
$ sudo cp w5500-overlay.dtb /boot/overlays/.

# config.txt 내용 추가
$ sudo vi /boot/config.txt
(내용 추가)
dtoverlay=w5500
```

설정 완료 후 RPI zero를 reboot 한다.
\$ sudo reboot

이제 거의 모든 설정이 끝났다. 컴파일 후 RPI zero로 복사했던 모듈을 활성화 시켜주면 된다.

```
$ sudo insmod w5100.ko
$ sudo insmod w5100-spi.ko
```

모듈이 정상적으로 올라가면 아래 그림과 같이 새로운 ethernet interface가 생성된다.

![new eth](/files/rpi-zero-with-wiz850io-new_eth.png)

새로 생성된 eth0를 보면 dt overlay에서 설정했던 mac address가 적용된 것을 볼 수 있다.

이 때 IP를 자동으로 받아오려면 DHCP를 지원하는 AP(공유기)와 연결된 상태여야 한다.

### 동작 확인 및 테스트

가장 간단한 동작 확인 방법은 ping 이다.

기존의 무선랜(wlan0) 인터페이스를 down 시키고, eth0만 활성화 된 상태로 ping을 수행해 본다.

같은 대역의 PC로 ping을 날려 보았다.

\$ ping 192.168.50.80

![new eth](/files/rpi-zero-with-wiz850io_ping.png)

ping이 잘 되는 것을 확인 할 수 있다.

---

여기까지 W5500기반의 WIZ850IO 모듈을 이용하여 RPI zero에 이더넷을 추가해 보았다.

다음 글에서 dts 파일의 SPI 속도를 변경해 보고, 변경이 잘 되었는지 iperf로 속도 변화 테스트를 진행한 내용을 정리해 보겠다.

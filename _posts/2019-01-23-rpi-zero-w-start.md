---
layout: post
title: "[RaspberryPi zero W] headless 초기 설정"
categories: RaspberryPi
date: 2019-01-23
tags: [raspberrypi, rpizero, headless, wlan]
comments: true
---

**Raspberry Pi zero W**(이하 RPI zero)를 사용해 볼 기회가 생겼다.

RPI zero 모델은 무선랜 지원 여부에 따라 두 타입이 있는데, 제품명에 W가 붙은 것이 무선랜 지원 모델이다. 이 글에서 RPI zero 초기 설정을 진행하면서 실패(?)한 경험을 정리하려고 한다.

제품은 이렇게 생겼다. (확장 헤더 부분은 따로 납땜을 해 줘야 했다.)

![Raspberry Pi Zero W](/files/rpizerow_board.jpg){: width="70%"}

## 환경 준비

### Material

- Micro sd card: 최소 4GB 이상
- SD card reader

### 환경

- Windows 10 64bit

* Raspbian image
  Raspbian은 두 버전의 커널을 비교 사용할 일이 있어서 다음 버전들을 사용했다.

  - 2017-11-29-raspbian-stretch-lite
  - 2018-03-13-raspbian-stretch-lite

- Raspbian 버전 정보

  - [Rasbian release note](http://downloads.raspberrypi.org/raspbian/release_notes.txt){:target="\_blank"}

- 이전 버전의 Raspbian 다운로드
  - [Raspbian image download](http://downloads.raspberrypi.org/raspbian/images/){:target="\_blank"}
  - [Raspbian Lite image download](http://ftp.jaist.ac.jp/pub/raspberrypi/raspbian_lite/images/){:target="\_blank"}

### Headless

Headless setup은 디스플레이 없이 설정하는 방식을 말한다. 단어 검색을 해보니 보통 IT 분야에서는 모니터가 없다는 뜻으로 쓰이는 듯 하다.

Raspberry pi 3 의 경우 USB와 같은 인터페이스가 충분해서 직접 모니터, 키보드, 마우스를 연결해서 설정해도 무방하지만 하나하나 선 연결하는게 굉장히 번거롭다.

특히나 RPI zero 모델은 USB 포트가 micro 5pin 1개 뿐이기 때문에 포트를 확장할 수 있는 허브가 없다면 모니터를 통한 설정은 어렵다고 봐야 한다.

여러 모로 headless 설정이 훨씬 간편하다고 볼 수 있다.

---

## Raspberry Pi zero W headless 설정

raspberry pi 3에서 사용했던 방법을 떠올려 그대로 설정하려고 하니 네트워크 연결 방식이 다르다는 것을 깨달았다.

Raspberry Pi zero W는 compact 한 사이즈로 이더넷 커넥터가 없고, 대신 무선랜(WiFi)을 지원한다.

그래서 검색을 해 보니, RPI zero W 제품을 위한 headless 설정 방법이 여럿 나왔다.

검색 결과를 가지고 다음과 같이 설정을 진행했다.

---

### Micro SD card에 raspbian 설치 (image write)

Micro SD card에 raspbian을 설치하는 상세 방법은 다음 글을 참조하자. (ssh 사용 설정 까지)

[[RaspberryPi3] 라즈베리파이3 - 라즈비안(Raspbian) OS 설치](https://inmile.tistory.com/6?category=793890){:target="\_blank"}

---

### 무선랜 설정을 위한 wpa_supplicant.conf 작성 - 실패한 내용 (참고)

wpa_supplicant.conf 라는 빈 파일을 만들고, 아래 내용을 작성한다.

_**동작하지 않았던 설정파일**_

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

이 파일 역시 빈 ssh 파일과 같은 위치(boot 파티션)에 넣어준다.

그리고 sd card를 RPI zero W에 삽입한 후 전원을 연결하면 된다.
초기 설정되는 시간을 감안해 약 90~120초 정도가 지난 후 raspberrypi.local로 ssh 연결을 해보라고 하는데...

**연결이 되지 않는다.**

혹시 뭔가 잘못 설정했나 싶어 이미지를 새로 올려보고 설정파일 내용을 바꿔보고 이것저것 다 해봤지만 여전히 안 된다.

딱히 복잡한 설정을 한 것도 아닌데 뭔가 문제가 있다. 다시 검색에 들어갔다.

일단 무선랜(wlan)을 통한 haedless 설정은 포기하고, 원인을 확인하기 위해서 확장 헤더 납땜 후 다음 사진과 같이 serial 케이블을 연결해 RPI zero에 접속했다.

serial을 사용하려면 boot의 config.txt파일에 아래 옵션을 추가해야 한다.

```
# Enable uart
enable_uart=1
```

![Raspberry Pi Zero W - serial 연결](/files/rpizerow_serial.jpg){: width="70%"}

접속 후 raspi-config으로 wifi 설정을 시도하니 아래와 같은 메시지가 연달아 나왔다. 무선랜 설정 쪽에 문제가 있어 보였다.

```
Could not communicate with wpa_supplicant.
There was an error running option N2 Wi-fi.
```

---

### 무선랜 설정을 위한 wpa_supplicant.conf 작성 - 성공!

시리얼을 통해 무선랜 쪽에 문제가 있는걸 확인하고 검색하던 중, wpa_supplicant 디버깅에 관한 내용이 눈에 띄었다.

다음 링크에 보면 wpa_supplicant 명령어에 대한 내용이 있다.

[Raspberrypi forum: Raspbian Stretch: Wifi not starting on boot](https://www.raspberrypi.org/forums/viewtopic.php?t=191061&start=25){:target="\_blank"}

RPI zero에 시리얼로 접속한 상태에서 아래 명령을 수행해보니, 에러가 출력되었다.

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

첫번째 문제는 ssid와 password를 설정하는 network 설정 부분에서 공백(space)을 인식하지 못하는 것이었다.

공백을 탭으로 수정한 후 다시 명령 수행.

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

두번째 문제는 'GROUP=netdev' 옵션을 인식하지 못하는 것이었다.

wpa_supplicant.conf 파일 관련해서 참고했던 포스팅 날짜가 2017년 초 버전이었는데, Raspbian 버전이 올라가면서 뭔가 바뀐 것일까?

이 부분은 확인해봐야 할 것 같다.

아무튼, 해당 옵션을 제거 후 다시 돌려보니 잘 된다.

디버깅을 통해 만든 최종 설정 파일은 다음과 같다.

#### wpa_supplicant.conf

(network 부분은 tab으로 작성되어야 함. 에디터 프로그램으로 작성 시 주의)

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

위 설정파일로 적용해보니 잘 된다.

## 요약

과정을 쓰다보니 길어졌지만, 요약하면 다음과 같다.

- 준비한 micro sd card에 raspbian image write
- boot 드라이브에 빈 ssh 파일 및 [wpa_supplicant.conf](#wpa_supplicant.conf) 작성하여 추가
- sd card를 RPI zero에 삽입하고 전원 연결 후 1~2분 대기
- raspberrypi.local로 ssh 접속 (초기 계정: pi / raspberry)
  - 접속 전에 먼저 ping을 해봐도 좋다. (\$ ping raspberrypi.local)

혹시라도 다른 raspberry pi가 연결되어있으면 그쪽으로 연결될 수 있으므로 단독 연결하는게 좋다.

초기 설정은 이걸로 끝이다.

다음 글에서 WIZ850IO 모듈을 이용해 이더넷을 추가하는 방법에 대해 정리할 예정이다.

---

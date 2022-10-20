---
layout: single
title: "W5100S-EVB-Pico에 M5Stack 센서 연결하여 사용하기 with Arduino IDE"
categories: w5100s-evb-pico
date: 2022-05-02
tags: [W5100S-EVB-Pico, m5stack, unit, tcs34725]
---

# 개요

W5100S-EVB-Pico에 M5Stack Sensor Unit을 연결하고 Arduino IDE를 사용하여 동작시키는 방법에 대해 정리해보고자 한다.

사무실에 활용 가능한 M5Stack Unit 센서들이 있어서 이를 W5100S-EVB-Pico에 연결하여 활용해보기로 한 것이 시작이다.

M5Stack에서는 Unit 형태로 많은 센서 종류를 지원하고 있는데, 그 중 color 센서(TCS34725)를 사용하여 테스트를 진행했다.


# 하드웨어

메인 디바이스로 W5100S-EVB-Pico를 사용하고, Color 센서 유닛과 연결 케이블을 사용했다.

- [W5100S-EVB-Pico (H/W v1.0)](https://docs.wiznet.io/Product/iEthernet/W5100S/w5100s-evb-pico)
- [M5Stack Color sensor unit](https://docs.m5stack.com/en/unit/color)
- Grove to Jumper cable


## Pin 연결


Pin 정보는 위 하드웨어 정보 링크에서 참조할 수 있다.

Color 센서 유닛은 I2C 인터페이스로 동작하는데, 아래와 같이 Grove 인터페이스 형태로 되어 있다. 

(이미지 출처: https://docs.m5stack.com/en/unit/color)

<img src="https://static-cdn.m5stack.com/resource/docs/products/unit/color/color_01.webp" width="40%">

Pin 형태인 W5100S-EVB-Pico 보드와 어떻게 연결할지 막막하던 중 다행히 Grove to Jumper 케이블을 하나 찾아서 연결할 수 있었다. (Male-Female이 맞지 않아서 점퍼 케이블을 4개 추가로 사용했다.)

Grove to Jumper 키워드로 검색해보니 여러 형태로 판매하고 있었다. 추가로 필요하면 구입해서 사용하면 될 듯 하다.

다음과 같이 Pin 연결을 해주었다.

| [W5100S-EVB-Pico] | [Color sensor(TCS34725)] |
| ----------------- | ------------------------ |
| 3V3 (Pin 36)      | 5V                       |
| GND (Pin 8)       | GND                      |
| GPIO5/SCL (Pin 7) | SCL                      |
| GPIO4/SDA (Pin 6) | SDA                      |

<!-- 연결 사진 -->

아래는 연결을 완료한 모습이다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-connection.jpg?raw=true" />

하드웨어 준비는 이렇게 완료했다.


# Arduino IDE 사용하기

## 보드 매니저 설정

Arduino IDE에서 W5100S-EVB-Pico를 사용하기 위해 다음 글을 참조했다.

- [Arduino Raspberry Pi Pico/RP2040 Ethernet: W5100S-EVB-Pico](https://www.hackster.io/loveivyou/arduino-raspberry-pi-pico-rp2040-ethernet-w5100s-evb-pico-12de75)

간단한 센서 테스트만 진행할 것이기 때문에 이더넷 연결 부분은 진행하지 않았다.

과정을 요약하면 다음과 같다.

Arduino IDE 실행 후, `파일` - `환경설정` 에서 추가적인 보드 매니저 URLs에 아래 URL을 추가하고, 확인 버튼을 눌러 저장한다.

```
https://github.com/WIZnet-ArduinoEthernet/arduino-pico/releases/download/global/package_rp2040-ethernet_index.json
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-board_manager_url.png?raw=true" />

툴 - 보드 - 보드 매니저 메뉴로 진입 후 `pico` 키워드로 검색 후 스크롤을 내려보면, `Raspberry Pi Pico/RP2040 Ethernet` 패키지를 찾을 수 있다.

>2022년 5월 기준, 최신 버전은 1.0.1 이다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-board_manager.png?raw=true" />

설치를 진행한 뒤, `WIZnet W5100S-EVB-Pico` 보드 선택까지 진행한다.

- 툴 > 보드 > Raspberry Pi RP2040 Boards > WIZnet W5100S-EVB-Pico 선택

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-select_board.png?raw=true" />


## TCS34725 

### 라이브러리 설치

Adafruit에서 TCS34725의 아두이노 라이브러리를 지원한다.

Arduino IDE 메뉴에서 `툴` > `라이브러리 매니저` 선택(또는 단축키 `Ctrl+Shift+i`) 후, tcs34725 키워드로 필터링한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-lib.png?raw=true" />


`Adafruit TCS34725`를 선택 후 설치한다.



### 샘플 코드

Adafruit 센서의 샘플 코드 중, tcs34725 샘플을 그대로 사용했다.
- 참조: https://github.com/adafruit/Adafruit_TCS34725/tree/master/examples/tcs34725

아래 메뉴를 통해 설치된 라이브러리에 대한 샘플 코드를 바로 불러올 수 있다.

- 파일 > 예제 > Adafruit TCS34725 > tcs34725 선택


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-sample.png?raw=true" />



### 업로드

Arduino IDE를 사용하여 W5100S-EVB-Pico를 포함한 Raspberry Pi Pico 보드에 **최초로 업로드를 할 때**에는 Boot 모드로의 변경이 필요하다.

한번 업로드를 하고 나면, 이후에는 모드 변경 없이 **인식된 Com Port를 통해 바로 업로드가 가능**하다.

W5100S-EVB-Pico H/W v1.1 부터는 전원을 다시 인가하지 않아도 2개 버튼(RUN, BOOTSET)을 눌러 모드를 변경할 수 있다.

이에 대한 히스토리와 사용법은 아래 글에서 참조할 수 있다.

<!-- v1.1 관련글 링크 -->
- [Added reset button to W5100S-EVB-Pico](https://www.hackster.io/scott_jeong/added-reset-button-to-w5100s-evb-pico-ef87f0)

내가 가진 보드는 H/W v1.0이고, 이 경우 BOOTSEL 버튼을 누른 상태로 전원을 인가하면 된다.

모드가 변경되면 PC에서 RPI-RP2 라는 이름의 이동식 디스크가 나타나고, 여기에 펌웨어를 올릴 수 있는 상태가 된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-rpdisk.png?raw=true" />

이제 업로드 버튼을 눌러 펌웨어를 업로드한다. 컴파일 후 업로드가 진행된다.

업로드가 완료되면 아래와 같은 로그를 볼 수 있다. 

>로그가 보이지 않는다면 파일 > 환경설정에서 `다음 동작중 자세한 출력 보이기`의 컴파일, 업로드 항목을 체크한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-upload_log.png?raw=true" />


### 모니터링

Arduino IDE의 시리얼 모니터 메뉴(단축키: Ctrl + Shift + m)를 통해 센서 데이터가 올라오는 걸 확인한다.

센서부를 움직이면 Color 측정값이 변하는 것을 볼 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_m5sensor-monitor.png?raw=true" />


여기까지 W5100S-EVB-Pico와 Arduino IDE를 사용하여 센서 동작을 확인해 보았다.

다음으로는 다른 종류의 센서도 동작을 테스트해 보고, 클라우드로 전송하는 등의 기능을 추가해서 확장해볼 예정이다.

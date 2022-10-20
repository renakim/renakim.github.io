---
# layout: single
title: "W5100S-EVB-Pico에서 Micropython 및 MQTT를 사용하여 Azure IoT Hub에 연결하기"
categories: w5100s-evb-pico
date: 2022-09-06
tags: [w5100s-evb-pico, micropython, azure, iot hub, rp2040, w5100s, wiznet]
---

Micropython을 사용하여 W5100S-EVB-Pico를 Azure IoT Hub에 MQTT로 연결하고 메시지를 송수신하는 과정에 대해 설명한다.

IoT Hub 인증 방식은 SaS Token을 사용했다.

## 준비

**H/W**
* W5100S-EVB-Pico
* Micro 5pin USB cable
* LAN cable

**S/W**
* Thonny
  * RP2040 Micropython 개발환경
* [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases)
  * 장치 정보 확인
  * 데이터 모니터링
  * C2D 전송



# Azure 리소스 준비

## Azure IoT Hub 생성

Azure IoT Hub를 생성하는 방법은 Azure Portal, Azure CLI, REST API 등 다양하다. 처음에는 주로 Azure Portal을 통해 생성하는 방법을 사용한다.

방법은 아래 링크에서 참조할 수 있다.

- [Azure Portal을 사용하여 IoT Hub 만들기](https://docs.microsoft.com/ko-kr/azure/iot-hub/iot-hub-create-through-portal)



# Micropython 펌웨어

## 빌드

빌드 작업은 WSL2 (Ubuntu 20.04.4 LTS) 환경을 사용했다.

```
rena@Rena-PC:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.4 LTS
Release:        20.04
Codename:       focal
```

>Tool 설치 등 빌드 환경 구축에 대한 내용은 [Micropython의 공식 문서](https://docs.micropython.org/en/latest/develop/gettingstarted.html#compile-and-build-the-code)에서 참조할 수 있다.

빌드 과정은 Micropython Repositroy 내 README에서 참조했다.

* https://github.com/micropython/micropython/tree/master/ports/rp2


### Repository clone

Submodule을 포함하여 Repository를 clone 하고, submoule들을 받아온다.

<!-- git clone --recurse-submodules https://github.com/micropython/micropython.git -->

```
git clone https://github.com/micropython/micropython.git
cd micropython

git submodule update --init
```


### 서브모듈 빌드
```
make -C ports/rp2 submodules
```

### mpy-cross 빌드 (MicroPython cross-compiler)

장치 펌웨어를 빌드하기 전에 mpy-cross 빌드가 선행되어야 한다.

```
make -C mpy-cross
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-01.png?raw=true" />



### W5100S-EVB-Pico 디바이스 펌웨어 빌드

지원되는 장치 중 `W5100S_EVB_PICO`를 사용하여 펌웨어를 빌드한다.

>지원 목록은 [Micropython: ports/rp2/boards](https://github.com/micropython/micropython/tree/master/ports/rp2/boards)에서 확인할 수 있다.

```
cd ports/rp2
make BOARD=W5100S_EVB_PICO submodules
make BOARD=W5100S_EVB_PICO
```

마지막 빌드 과정이다. 시간이 최소 몇 분 이상 소요된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-02.png?raw=true" />



## 펌웨어 업로드

빌드가 완료된 펌웨어를 장치로 업로드한다.

### Boot 모드 진입

H/W v1.0과 v1.1의 모양이 약간 다른데, 내가 가지고 있는 보드는 v1.0 이어서 보드의 BOOTSEL 버튼을 누른채로 전원을 인가하면 Boot 모드로 진입된다.

v1.1의 경우 BOOTSEL 버튼을 누른 채로 RUN 버튼을 누르면 Boot 모드로 진입되어 전원을 다시 인가할 필요가 없다.

### 펌웨어 업로드

빌드된 펌웨어는 다음 경로에 위치하고 있다.

- `micropython/ports/rp2/build-W5100S_EVB_PICO`

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-03.png?raw=true" />

이 중 `firmware.uf2` 파일을 업로드한다.


이제 펌웨어 작업은 끝났다. 

다음은 Thonny로 라이브러리 설치, 코드 작성 및 업로드를 하고 Azure IoT Expolrer를 사용하여 데이터 송수신 및 모니터링을 진행하면 된다.



# 디바이스 코드 작성


예제 코드는 Azure-Samples의 IoTMQTTSample 코드에서 참조했다.

- [Azure-Samples/IoTMQTTSample - src/MicroPython](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/MicroPython)

## IoT Explorer에서 디바이스 정보 가져오기

Azure IoT Hub에 연결하기 위한 정보를 얻어 코드에 작성해야 한다.

>MQTT로 통신할 때 각 필드에 요구되는 내용은 아래 가이드 문서에서 확인
>https://docs.microsoft.com/ko-kr/azure/iot-hub/iot-hub-mqtt-support#using-the-mqtt-protocol-directly-as-a-device

예제 코드의 경우는 아래 데이터를 얻어와야 한다.

* Device Connection String
* Device SAS Token

>코드를 보면 Connection String을 파싱해 Host name, Device Id, Shared access key 값을 얻도록 구현되어 있다.

정보를 가져오는 방법 역시 여러가지인데, 그 중 IoT Explorer을 사용했다.

- [Azure IoT Hub Release](https://github.com/Azure/azure-iot-explorer/releases)


### IoT Explorer IoT Hub 연결 설정

* 참조: https://docs.microsoft.com/ko-kr/azure/iot-fundamentals/howto-use-iot-explorer#connect-to-your-hub

우선, IoT Explorer에서 내 IoT Hub에 접근할 수 있도록 액세스 권한을 부여해야 한다.

기본 설정된 권한들 중에서 모든 권한을 포함하는 iothubowner 권한을 IoT Explorer에 부여해 줄 것이다.

iothubonwer를 클릭하고, Primary connection string 부분 우측의 버튼을 클릭해 값을 복사한 다음, IoT Explorer에서 Add connection을 클릭했을 때 나오는 창에 붙여넣고 Save 한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-04_0.png?raw=true" />


초기에 이 설정을 한번만 해주면, IoT Hub와 디바이스에 대한 대부분의 작업을 Tool에서 수행할 수 있다.



디바이스를 생성한 다음, 그림과 같이 정보값을 가져온다.

**디바이스 생성**

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-04.png?raw=true" />


**Connection String 복사**
<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-05.png?raw=true" />


**SAS Token 생성 및 복사**

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-06.png?raw=true" />




## 전체 코드

코드는 아래 링크에 업로드 해 두었다.

- https://github.com/renakim/W5100S-EVB-Pico-Micropython/tree/main/examples/Azure


원본 예제 코드에서 아래 내용을 추가, 수정했다.

* W5100S 네트워크 연결 설정
* Telemetry 메시지 전송 형태 수정
  * String -> Json string

Json으로 변경하지 않으면 IoT Explorer에서 데이터를 식별하기가 어렵다.




## 라이브러리 설치

Thonny 환경에서 라이브러리를 설치한다.

상단 메뉴의 Tools - Manage packages 선택 후, umqtt를 입력한 다음 검색한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-07.png?raw=true" />

검색된 패키지들 중 umqtt.simple과 umqtt.robust를 차례로 설치한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-08.png?raw=true" />

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-09.png?raw=true" />


설치가 잘 되었다면 좌측 목록에서 umqtt를 클릭했을 때 다음과 같이 설치된 패키지가 나온다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-10.png?raw=true" />


## 실행 및 모니터링


### Telemetry

```
connecting
Publishing
Sending message 0
Sending message 1
Sending message 2
Sending message 3
Sending message 4
Sending message 5
Sending message 6
Sending message 7
Sending message 8
Sending message 9
Sending message 10
waiting for message
Received message
b'message from IoT Hub'
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-11.png?raw=true" />


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-12.png?raw=true" />


### C2D message

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-13.png?raw=true" />

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-14.png?raw=true" />




# Reference

* [RP2040-HAT-MicroPython Repository](https://github.com/Wiznet/RP2040-HAT-MicroPython)
* [Azure IoTMQTTSample - Micropython](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/MicroPython)
* [Micropython forum post from nickehallgren](https://forum.micropython.org/viewtopic.php?t=12955&p=70605)

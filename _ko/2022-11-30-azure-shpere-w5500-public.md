---
layout: single
title: "Azure Sphere: W5500을 이더넷 어댑터로 사용하기"
categories: azure-sphere
date: 2022-11-30
tags: [azure sphere, w5500, azure, iot central, public network, wiznet]
---


## 개요

최근 Azure Sphere의 10/100 이더넷 네트워크 인터페이스 어댑터로 WIZnet W5500이 채택되었고, 22.09 릴리즈 버전에서 업데이트 되었다.

아래 링크에서 22.09 릴리즈에 대한 업데이트 내용을 확인할 수 있다.

- [Azure Sphere 새 기능: 22.09 릴리즈](https://learn.microsoft.com/ko-kr/azure-sphere/product-overview/whats-new#whats-new-in-the-2209-release)
- [General Availability: Azure Sphere version 22.09 new and updated features](https://techcommunity.microsoft.com/t5/internet-of-things-blog/general-availability-azure-sphere-version-22-09-new-and-updated/ba-p/3638460)


Avnet Azure Sphere Starter Kit를 사용하여 연결 및 테스트가 가능하여 이에 대한 내용을 정리해 본다.

## 하드웨어 구성

- Avnet Azure Sphere Starter Kit V1
    - V2 사용 가능
- WIZ850io (W5500 ethernet module)
- Micro 5 pin USB cable
- LAN cable
- Jumper cables

### Pin 연결

점퍼 케이블을 사용하여 Avnet Azure Sphere Starter Kit V1 보드와 WIZ850io 모듈을 연결한다.


| Avnet Azure Sphere starter kit V1 | WIZ850io |
| --------------------------------- | -------- |
| GND                               | GND      |
| +3.3V                             | 3.3V     |
| SCK                               | SCLK     |
| MOSI                              | MOSI     |
| MISO                              | MISO     |
| RST                               | RSTn     |
| INT                               | INTn     |
| CS                                | SCNn     |


모든 핀을 연결하면 다음과 같은 모양이 된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere_starterkit-w5500.jpg?raw=true" />


## Board Config

### Board config imagepackage

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-2.png?raw=true" />

Azure Sphere SDK가 설치된 경로로 가면 Board Config Presets 폴더가 있고, 그 안에 Preset 파일들이 들어 있다.

이 중 `lan-w5500-isu1-int2.imagepackage` 파일을 사용한다.

Starter Kit V2를 사용하는 경우는, lan-w5500-isu0-int5.imagepackge 파일을 사용할 수 있다.

| 참조: [https://learn.microsoft.com/ko-kr/azure-sphere/network/ethernet-adapters#wiznet-w5500-based-adapters](https://learn.microsoft.com/ko-kr/azure-sphere/network/ethernet-adapters#wiznet-w5500-based-adapters)

아래 명령을 사용해 imagepackage를 디바이스로 업로드한다.

이미지를 올리면 디바이스가 자동 Reset 되고, 다음과 같이 설치된 이미지를 확인할 수 있다.

```bash
azsphere device sideload deploy --image-package lan-w5500-isu1-int2.imagepackage

azsphere device image list-installed
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-3.png?raw=true" />

### 네트워크 인터페이스 확인

Pin을 잘 연결했다면, 네트워크 인터페이스가 새로 추가된 것을 확인할 수 있다. (eth0)

```bash
azsphere device network list-interfaces
```


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-4.png?raw=true" />

(아래는 어플리케이션을 실행 후 캡쳐한 것이다.)

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-5.png?raw=true" />

## 네트워크 설정

네트워크 동작을 확인하고 검증하기 위해 2개의 샘플 어플리케이션을 실행하여 테스트를 진행했다.

## Sample Application Test #1: **PrivateNetworkServices**

[https://github.com/Azure/azure-sphere-samples/tree/main/Samples/PrivateNetworkServices](https://github.com/Azure/azure-sphere-samples/tree/main/Samples/PrivateNetworkServices)


### DHCP 사용 설정

API 참조: [https://learn.microsoft.com/ko-kr/azure-sphere/reference/applibs-reference/applibs-networking/function-networking-ipconfig-enabledynamicip](https://learn.microsoft.com/ko-kr/azure-sphere/reference/applibs-reference/applibs-networking/function-networking-ipconfig-enabledynamicip)

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-7.png?raw=true" />

Static 설정 대신, DHCP 설정 함수를 사용했다.

```c
Networking_IpConfig_EnableDynamicIp(&ipConfig);
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-8.png?raw=true" />

DHCP 서버로 설정하는 부분인데, 테스트에는 필요하지 않아 주석 처리했다.

### 실행

아래는 Device debug output 내용이다. 사용 가능한 네트워크 인터페이스 목록을 가져오고, 각 항목에 대해 상태를 읽어보는 부분이다.

interface #1 이 eth0 (이더넷)이고, 상태는 0x0f로 네트워크 및 인터넷에 정상 연결된 것을 의미한다.

상태를 나타내는 Hex 값은 비트 마스크로, [어플리케이션 라이브러리 참조 문서](https://learn.microsoft.com/ko-kr/azure-sphere/reference/applibs-reference/applibs-networking/enum-networking-interfaceconnectionstatus)에서 상세 내용을 확인할 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-9.png?raw=true" />

다시 커맨드로 네트워크 상태를 확인해 보면, DHCP 서버(공유기)를 통해 IP를 할당받아 인터넷에 연결된 상태임을 볼 수 있다.

```c
azsphere device network list-interfaces
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-10.png?raw=true" />

## Sample Application Test #2: Azure IoT

인터넷에 연결된 것을 확인한 후, Azure IoT 샘플을 사용하여 IoT Central에 연결해 보았다.

어플리케이션을 설정하는 과정은 공식 문서에 잘 나와있어서, 여기에서는 생략한다. 추후에 시간이 되면 정리할 생각이다.

- [azure-sphere-samples: AzureIoT: StartWithIoTCentral](https://github.com/renakim/azure-sphere-samples/blob/master/Samples/AzureIoT/READMEStartWithIoTCentral.md)
- [MS Guide: Azure Sphere에서 작동하도록 Azure IoT Central 설정](https://learn.microsoft.com/ko-kr/azure-sphere/app-development/setup-iot-central?tabs=cliv2beta)

과정을 요약하면 다음과 같다.

1. Azure IoT Central 어플리케이션 생성
2. 디바이스 등록 그룹 생성
    1. 테넌트 CA 인증서 다운로드 (with Azure Sphere CLI)
    2. CA 인증서 업로드 (자동 Verify)
3. AzureIoT 샘플 디렉토리 내 Tools 스크립트를 사용하여 Azure IoT Central endpoint 값 획득
4. app_manifest.json 내용 업데이트 후 실행

업데이트 된 app_manifest.json 내용은 다음과 같다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-11.png?raw=true" />

추가로, Telemetry 값을 전송하려면 main.c에서 Telemetry 전송 여부를 결정하는 변수를 true로 변경해야 한다. 

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-12.png?raw=true" />

이전 버전에서는 기본적으로 전송하도록 설정되어 있었는데 최근 변경된 듯 하다.

Telemetry 전송 주기는 telemtryPeriod 변수를 통해 설정할 수 있다. 기본 5초이고, 나는 30초로 변경했다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-13.png?raw=true" />

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-14.png?raw=true" />

이제 빌드 후 디버그를 실행(실행 버튼 클릭 또는, 단축키 `F5` 입력)하면, 디바이스 인증 및 연결이 진행되고, Telemetry가 전송된다.

아래는 정상 연결되었을 때 디바이스 디버그 출력 메시지 일부이다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-15.png?raw=true" />

아래 화면과 같이 IoT Central 디바이스 Raw Data 탭에서 실시간 데이터가 수신되는 것을 확인할 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/azsphere/azsphere-w5500-16.png?raw=true" />

W5500을 통한 네트워크 연결과 샘플 동작이 검증되었으니, 여러 응용에서 활용할 수 있을 것 같다. 🙂
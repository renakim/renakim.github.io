---
layout: single
title: "W5100S-EVB-Pico으로 AWS IoT Core에 연결하기 (with Micropython)"
categories: w5100s-evb-pico
date: 2022-10-14
tags: [w5100s-evb-pico, micropython, aws, iot core, rp2040, w5100s, wiznet]
---

W5100S-EVB-Pico 보드를 사용하여 AWS IoT Core에 연결해보자.

Micropython을 사용하여 구현하고, MQTT 프로토콜로 연결할 것이다.

다음 과정으로 진행한다.

* AWS 서비스
  * AWS IoT Core 및 디바이스 생성
  * 디바이스 인증서 다운로드
* Micropython 펌웨어 업로드
* 메인 코드 작성
  * 인증서 변환
* 동작 확인

## 준비

참고로 호스트 PC 환경은 Windows 10 64bit를 사용하고, WSL2 환경을 함께 사용했다. (OpenSSL 사용)

**H/W**
* W5100S-EVB-Pico
* Micro 5pin USB
* LAN cable

**S/W or Platform**

* Thonny
  * Micropython 개발환경
* AWS IoT


# AWS 서비스

AWS 서비스에서 디바이스를 생성하는 방법은 외부 컨텐츠를 링크하였다.

C 기반으로 WizFi360-EVB-Pico를 AWS IoT Core에 연동하는 과정을 정리한 컨텐츠인데, AWS IoT에서 디바이스를 생성하는 과정이 잘 설명되어 있다.

* [Connect WizFi360-EVB-Pico to AWS IoT Core](https://maker.wiznet.io/mason/projects/connect-wizfi360-evb-pico-to-aws-iot-core/)


전체 기능에 대한 설명은 공식 가이드 문서에서 참조할 수 있다.

* [AWS IoT 리소스 만들기](https://docs.aws.amazon.com/ko_kr/iot/latest/developerguide/create-iot-resources.html)

디바이스 생성을 완료했다면, 디바이스 인증서와 루트 인증서를 받을 수 있을 것이다.

가이드에도 강조되어 있지만 디바이스 **생성 시 나타나는 인증서 다운로드 페이지에서 인증서를 저장**해야 한다. 그렇지 않으면 사물 생성 과정을 다시 진행해야 한다.

여기에서는 디바이스 인증서와 Pricate Key만 사용한다.


# Micropython 펌웨어 업로드

펌웨어를 빌드하고 업로드 하는 과정은 기존 Azure 연동 과정을 다룬 컨텐츠를 대신 링크한다.

- (한글) [W5100S-EVB-Pico에서 Micropython 및 MQTT를 사용하여 Azure IoT Hub에 연결하기](https://inmile.tistory.com/46)
- (영문) [Connect to Azure IoT Hub using Micropython on W5100S-EVB-Pico](https://maker.wiznet.io/rena/projects/connect-to-azure-iot-hub-using-micropython-on-w5100s-evb-pico/)





# 디바이스 코드 작성


여기에서, 다음 변수들을 수정해야 한다.

* Device Id
* MQTT Topic
  * 지정된 토픽 사용
  * 또는 자유 토픽
* Hostname (Endpoint)
* 디바이스 인증서 설정
  * Device certificate
  * Device private key


전체 예제 코드는 아래 링크에서 참조할 수 있다.

- https://github.com/renakim/W5100S-EVB-Pico-Micropython/tree/main/examples/AWS


## 변수 작성

AWS에서 생성한 디바이스 리소스 정보에 따라 아래 변수 값을 입력한다.

토픽은 자유 토픽으로 사용하거나, 미리 정의된 토픽을 사용할 수 있다. 여기에서는 지정된 토픽을 사용하므로 따로 지정하지 않았다.

```python
device_id = '<Device Id>'
hostname = '<Hostname or Endpoint>'
mqtt_topic = f'$aws/things/{device_id}/shadow/update'
```


* Device Id: 생성한 Thing Name
* Hostname (Endpoint): AWS IoT - Settings에서 확인

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-aws-endpoint.png?raw=true" />


## 디바이스 인증서 설정

AWS 리소스 구성 부분에서 다운 받은 인증서를 W5100S-EVB-Pico로 업로드 하고, 경로를 설정한다.

다운로드 받은 인증서는 다음과 같은 형태일 것이다.

* \<Certificate Id>-certificate.pem.crt
* \<Certificate Id>-private.pem.key
* \<Certificate Id>-public.pem.key
* AmazonRootCA1.pem
* AmazonRootCA3.pem

이 중, certificate.pem.crt와 private.pem.key를 사용한다.

### 인증서 변환

Micropython 라이브러리에서 인식할 수 있는 형태로 인증서를 변환해야 한다.

다운받은 인증서는 base64 기반의 PEM 형태이고, 이를 바이너리 형태인 .der로 변환할 것이다.

>인증서 변환에는 OpenSSL을 사용하며, Windows 10 이상의 환경이라면 WSL(Windows Subsystem Linux) 환경을 활용하는 것을 추천한다.

아래 명령을 사용하여 디바이스 인증서와 Private key를 der 형식으로 변환한다.

```shell
# 디바이스 인증서 변환
openssl x509 -outform der -in certificate.pem.crt -out certificate.crt.der
# Key 변환
openssl rsa -inform pem -in private.pem.key -outform DER -out private.key.der
```


### 인증서 업로드

변환한 인증서를 Pico로 업로드하고, 경로 설정을 한다.

예제를 Clone 했다면 cert 라는 디렉토리가 있을 것이고, 여기에 .der로 변환한 디바이스 인증서와 key 파일을 복사 후 그 폴더를 Pico로 업로드 하면 된다.

>View - Files 옵션으로 좌측 탐색기 창을 사용할 수 있다.
><img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-aws-thonny_files.png?raw=true" />

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-aws-upload_cert.png?raw=true" />

cert 디렉토리에 커서를 대고 마우스 오른쪽 클릭 후, `Upload to /` 메뉴를 선택하면 폴더 그대로 업로드 할 수 있다.

경로나 파일명을 다르게 했다면 `main.py`에서 인증서 경로를 수정하면 된다.

```py
# Certificate path
cert_file = 'cert/certificate.crt.der'
key_file = 'cert/privateKey.key.der'
```


## Micropython 라이브러리 설치

Thonny 환경에서 라이브러리를 설치한다.

상단 메뉴의 Tools - Manage packages 선택 후, umqtt를 입력한 다음 검색한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-07.png?raw=true" />

검색된 패키지들 중 umqtt.simple을 선택하여 설치한다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-08.png?raw=true" />

설치가 잘 되었다면 좌측 목록에서 umqtt를 클릭했을 때 설치된 패키지를 볼 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-10.png?raw=true" />



# 동작 결과

## Application

Thonny에서 main.py를 띄운 채로 상단의 재생 모양 버튼을 클릭하거나, F5를 클릭하여 어플리케이션을 실행한다.

디버그 메시지가 다음과 같이 출력된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-thonny_mqtt.png?raw=true" />

>연결 에러가 발생한다면 인증서 문제일 가능성이 높다. 경로를 잘 설정했는지, .der로 변환한 인증서를 넣었는지 확인한다.


## AWS IoT 모니터링

AWS IoT 서비스 내 MQTT test client를 사용하면 디바이스 데이터 모니터링을 할 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-aws-mqtt_test_client.png?raw=true" />

Subscribe to a topic 탭에서 설정한 토픽 입력 후 Subscribe 버튼을 누르면 MQTT 메시지 구독이 시작되고, 디바이스로부터 전송되는 메시지를 모니터링 할 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-aws-monitoring.png?raw=true" />


여기까지 W5100S-EVB-Pico 디바이스를 사용하여 Micropython을 사용해 AWS IoT Core에 연결하는 방법을 정리했다.


# Reference

* [Connect to Azure IoT Hub using Micropython on W5100S-EVB-Pico](https://maker.wiznet.io/rena/projects/connect-to-azure-iot-hub-using-micropython-on-w5100s-evb-pico/)
* [RP2040-HAT-MicroPython Repository](https://github.com/Wiznet/RP2040-HAT-MicroPython)
* [Micropython forum post from nickehallgren](https://forum.micropython.org/viewtopic.php?t=12955&p=70605)


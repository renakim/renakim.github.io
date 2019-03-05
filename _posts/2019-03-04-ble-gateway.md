---
layout: posts
title: "IoT를 위한 BLE gateway 프로젝트 살펴보기"
categories: IoT
date: 2019-03-04
tags: [ble, gateway, wiznet, iot]
---

## Internet of Things

IoT(Internet of Things), 즉 사물 인터넷은 위키백과에 따르면 '각종 사물에 센서와 통신 기능을 내장하여 인터넷에 연결하는 기술, 즉 무선 통신을 통해 각종 사물을 연결하는 기술' 이다.

- [위키백과: 사물인터넷](https://ko.wikipedia.org/wiki/%EC%82%AC%EB%AC%BC%EC%9D%B8%ED%84%B0%EB%84%B7)

기본적으로 사물마다 내장된 센서들로부터 데이터를 수집하고, 수집한 데이터를 저장하며 나아가 클라우드 연동까지 하게 되는데
**데이터를 수집**하기 위해서는 사물(Things)로부터 데이터를 받아 인터넷으로 보내줄 수 있는 게이트웨이(gateway)가 필수적이다.

본 글에서는 대표적인 근거리 무선 통신 기술인 BLE 기반의 센서를 위한 게이트웨이 제품들과 프로젝트를 살펴보고자 한다. (BLE는 Bluetooth Low Energy의 약자로 Bluetooth 4.0 이후로 배포되는 저전력 블루투스 기술을 뜻한다.)

---

## Products

### BLE to TCP Gateway

WIZnet의 BLE to TCP 솔루션으로, 블루투스 장치가 인터넷 망에 접속하여 클라우드 서비스들에 연동될 수 있도록 만들어졌다.
WIZnet의 Hardware TCP/IP Chip인 W5500과 Wi-Fi 모듈인 WizFi310, 그리고 Nordic의 Coretex-M4F 기반의 BLE SoC인 nRF52832로 구성된 IoT gateway이다.

<img src="http://wiznetacademy.com/wp/wp-content/uploads/2016/10/ble_tcp_5.jpg">

자세한 내용은 아래 링크에서 확인할 수 있다.

- [WIZnet BLE to TCP Gateway](https://wiznetmuseum.com/portfolio-items/2-ble-to-tcp-gateway/)

### iNode series

BLE gateway 기능을 제공하는 iNode의 LAN 시리즈를 소개한다.

#### iNode LAN & iNode LAN Duos

<img src="https://wiznetmuseum.com/wp/wp-content/uploads/2019/02/iNode_LAN_71_1200.jpg">

iNode LAN은 이더넷 프로토콜을 사용하는 네트워크에서 iNode Bluetooth BLE 장치에 대한 액세스를 제공한다. 이 모듈을 사용하면 건물 내 iNode Care Sensor 센서의 범위를 늘리거나 iNode Nav의 움직임을 온라인으로 추적 할 수 있다. 

iNode LAN Duo 또한 IP 프로토콜 네트워크(LAN, WiFi 및 인터넷)에서 BLE 장치(Bluetooth Smart, IoT-Internet of Things)에 대한 엑세스를 제공하며, Bluetooth 2.1 리시버를 포함하고 있는것이 특징이다.

iNode LAN 시리즈는 모두 web configuration 환경을 제공한다. Bluetooth 모드 설정, 네트워크 설정, 원격 펌웨어 업데이트 등을 지원한다.

추가 정보는 아래 링크를 통해 확인하자.

- [iNode LAN introduction](https://wiznetmuseum.com/portfolio-items/inode-lan-bluetooth-gateway/)


#### iNode LAN Camera

<img src="https://inode.pl/images/inode/0-1000/iNode-LAN-Camera_%5B181%5D_1200.jpg" width="300">

iNode LAN 카메라는 5MPx 카메라와 JPEG compression 및 BLE의 조합으로 구성되어 있다. iNode LAN Camera와 iNode sensor 시리즈를 연동하여 움직임을 감지할 때 사진을 찍거나 경보음을 울리는 등의 동작을 할 수 있다.

---

## Related project

### BLE to Ethernet Thin Gateway

BLE to Ethernet Thin Gateway는 BLE(Bluetooth Low Energy)데이터를 이더넷을 통해 클라우드 서버로 전송하는 데이터 수집 프로젝트이다.

<img src="https://wiznetmuseum.com/wp/wp-content/uploads/2016/02/ble_gateway-1.jpg">

- [View more: BLE to Ehternet Thin Gateway](https://wiznetmuseum.com/portfolio-items/ble-to-ethernet-thin-gateway/)

https://wiznetmuseum.com/portfolio-items/ble-to-ethernet-thin-gateway/

### Telemedicine by using WIZwiki-W7500 and BLE

<img src="https://cdn.instructables.com/FJZ/OEAV/IB8J0RW2/FJZOEAVIB8J0RW2.LARGE.jpg?auto=webp&width=400">

- [View more: Telemedicine by using WIZwiki-W7500 and BLE](https://wiznetmuseum.com/portfolio-items/telemedicine-by-using-wizwiki-w7500-and-ble/)

---
layout: posts
title: "BLE gateway projects for IoT"
categories: IoT
date: 2019-03-04
tags: [ble, gateway, wiznet, iot]
---

[## BLE gateway

곳곳에 있는 센서들의 데이터를 수집하고 누적 저장하기 위해서는 게이트웨이가 필수적이다.

- BLE to TCP
  - BLE to ethernet
  - BLE to WiFi
-](## Internet of Things

IoT(Internet of Things), 즉 사물 인터넷은 위키백과에 따르면 '각종 사물에 센서와 통신 기능을 내장하여 인터넷에 연결하는 기술, 즉 무선 통신을 통해 각종 사물을 연결하는 기술' 이다.

- [위키백과: 사물인터넷](https://ko.wikipedia.org/wiki/%EC%82%AC%EB%AC%BC%EC%9D%B8%ED%84%B0%EB%84%B7)

기본적으로 사물마다 내장된 센서들로부터 데이터를 수집하고, 수집한 데이터를 저장하며 나아가 클라우드 연동까지 하게 되는데
데이터를 수집하기 위해서는 사물(Things)로부터 데이터를 받아 인터넷으로 보내줄 수 있는 게이트웨이(gateway)가 필수적이다.

본 글에서는 대표적인 근거리 무선 통신 기술인 BLE를 기반으로 한 제품들과 프로젝트를 살펴보고자 한다.

(BLE는 Bluetooth Low Energy의 약자로 Bluetooth 4.0 이후로 배포되는 저전력 블루투스 기술을 뜻한다.))

---

## Product

### BLE to TCP Gateway

WIZnet의 BLE to TCP 솔루션으로, 블루투스 장치가 인터넷 망에 접속하여 클라우드 서비스들에 연동될 수 있도록 만들어졌다.
WIZnet의 Hardware TCP/IP Chip인 W5500과 Wi-Fi 모듈인 WizFi310, 그리고 Nordic의 Coretex-M4F 기반의 BLE SoC인 nRF52832로 구성된 IoT gateway이다.

<img src="http://wiznetacademy.com/wp/wp-content/uploads/2016/10/ble_tcp_5.jpg">

자세한 내용은 아래 링크에서 확인할 수 있다.

- [WIZnet BLE to TCP Gateway](https://wiznetmuseum.com/portfolio-items/2-ble-to-tcp-gateway/)

### iNode series

iNode 시리즈 중 BLE gateway 기능을 하는 세 가지 제품을 소개한다.

#### iNode LAN

iNode LAN gives access to iNode Bluetooth BLE devices in networks with the Ethernet protocol. With this module you can increase the range of iNode Care Sensor sensors in the building or track on-line the movement of iNode Nav.

- CSR 1011
- W5500

- Support web configuration

<img src="https://wiznetmuseum.com/wp/wp-content/uploads/2019/02/iNode_LAN_71_1200.jpg">

- [iNode LAN introduction](https://wiznetmuseum.com/portfolio-items/inode-lan-bluetooth-gateway/)

#### iNode LAN Duos

iNode LAN Duos allows the existence of BLE devices (Bluetooth Smart, IoT-Internet of Things) in the IP protocol networks: LAN, Wi-Fi and the Internet. Additionaly it contains a BT2.1 receiver.

#### iNode LAN Camera

<img src="">

iNode LAN Camera is a combination of 5MPx camera with JPEG compression and with full BLE functionality (Bluetooth Smart, IoT:Internet of Things). Using the iNode LAN Camera you can easily associate the user identifier iNode Beacon with a recorded image (access control) or start recording when for example iNode Care Sensor #1 raises alarm associated with the opening of the door to the room or sensor iNode Care #5 detects vehicle in front of the gate.

- CSR 1011
- W5500
- OV5642

### Seeed Arch Link Board

<img src="https://os.mbed.com/media/cache/platforms/Arch_Link_rQW64G4.jpg.250x250_q85.jpg">

Arch Link is an mbed enabled development board based on Nordic nRF51822 and WIZnet W5500 ethernet interface. With Arduino form factor, Grove connectors and micro SD interface, it is extremely easy to create a bluetooth low energy device.

---

## Related project

### BLE to Ethernet Thin Gateway

BLE to Ethernet Thin Gateway is the data collecor to transmit the BLE(Bluetooth Low Eneregy) data to a cloud server through Ethernet.

- Nordic nRF51822
  - Multi-protocol Bluetooth® 4.0 low energy/2.4GHz RF SoC
- WIZnet W5500
  - Supports following Hardwired TCP/IP Protocols : TCP, UDP, ICMP, IPv4, ARP, IGMP, PPPoE

<img src="https://wiznetmuseum.com/wp/wp-content/uploads/2016/02/ble_gateway-1.jpg">

- [View more: BLE to Ehternet Thin Gateway](https://wiznetmuseum.com/portfolio-items/ble-to-ethernet-thin-gateway/)

https://wiznetmuseum.com/portfolio-items/ble-to-ethernet-thin-gateway/

### Telemedicine by using WIZwiki-W7500 and BLE

<img src="https://cdn.instructables.com/FJZ/OEAV/IB8J0RW2/FJZOEAVIB8J0RW2.LARGE.jpg?auto=webp&width=400">

- [View more: Telemedicine by using WIZwiki-W7500 and BLE](https://wiznetmuseum.com/portfolio-items/telemedicine-by-using-wizwiki-w7500-and-ble/)

---

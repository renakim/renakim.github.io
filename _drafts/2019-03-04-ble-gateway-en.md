---
layout: posts
title: "BLE gateway projects for IoT"
categories: IoT
date: 2019-03-04
tags: [ble, gateway, wiznet, iot]
---

## Internet of Things

According to Wikipedia, Internet of things is 'the network of devices such as vehicles, and home appliances that contain electronics, software, sensors, actuators, and connectivity which allows these things to connect, interact and exchange data.'

- [Wikipedia: Internet of things](https://en.wikipedia.org/wiki/Internet_of_things)

Basically, it collects data from built-in sensors for each object, stores the collected data, and even links to the cloud,
**In order to collect** data, a gateway is necessary to receive data from things and send them to the Internet.

In this article, I will look at gateway products and projects for BLE-based sensors, a typical short-range wireless communication technology. (BLE stands for Bluetooth Low Energy, which means low-power Bluetooth technology deployed since Bluetooth 4.0.)

---

## Products

### BLE to TCP Gateway

This is WIZnet's BLE to TCP solution enables Bluetooth devices to connect to the Internet and work with cloud services.
It is an IoT gateway composed of WIZnet hardware TCP/IP chip W5500, Wi-Fi module WizFi310, and Nordic Coretex-M4F based BLE SoC nRF52832.

<img src="http://wiznetacademy.com/wp/wp-content/uploads/2016/10/ble_tcp_5.jpg">

More information can be found at the link below.

- [WIZnet BLE to TCP Gateway](https://wiznetmuseum.com/portfolio-items/2-ble-to-tcp-gateway/)

### iNode series

There are three products in the iNode series that include the BLE gateway function.

#### iNode LAN & iNode LAN Duos

<img src="https://wiznetmuseum.com/wp/wp-content/uploads/2019/02/iNode_LAN_71_1200.jpg">

iNode LAN gives access to iNode Bluetooth BLE devices in networks with the Ethernet protocol. With this module you can increase the range of iNode Care Sensor sensors in the building or track on-line the movement of iNode Nav.

iNode LAN Duos allows the existence of BLE devices (Bluetooth Smart, IoT-Internet of Things) in the IP protocol networks: LAN, Wi-Fi and the Internet. Additionaly it contains a BT2.1 receiver.

- CSR 1011
- W5500

- any web browser
- built-in iNode Monitor (JavaScript/WebSocket/HTML5 application)


- [iNode LAN introduction](https://wiznetmuseum.com/portfolio-items/inode-lan-bluetooth-gateway/)


#### iNode LAN Camera

<img src="https://inode.pl/images/inode/0-1000/iNode-LAN-Camera_%5B181%5D_1200.jpg" width="300">

iNode LAN Camera is a combination of 5MPx camera with JPEG compression and with full BLE functionality (Bluetooth Smart, IoT:Internet of Things). Using the iNode LAN Camera you can easily associate the user identifier iNode Beacon with a recorded image (access control) or start recording when for example iNode Care Sensor #1 raises alarm associated with the opening of the door to the room or sensor iNode Care #5 detects vehicle in front of the gate.

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

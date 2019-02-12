---
layout: posts
title: "[RaspberryPi zero W] WIZ850io iperf Test"
categories: RaspberryPi
date: 2019-02-12
tags: [raspberrypi, rpizero, wiz850io, w5500, iperf]
---

지난 글에서, W5500 기반의 WIZ850io 모듈을 이용해 Raspberry pi zero W에 이더넷을 추가하는 내용을 진행했다.

본 글에서는 device tree의 SPI speed를 변경하면서 이더넷 속도 변화를 기록해볼까 한다. (WIZ850io는 SPI를 통해 연결된다.)

---
layout: posts
title: "[RaspberryPi zero W] 이더넷 추가 with WIZ850IO (2)"
categories: RaspberryPi
date: 2019-02-01
tags:
  - raspberrypi
  - 라즈베리파이
  - wiz850io
  - w5500
---

---

이전 글 [[RaspberryPi zero W] 이더넷 추가 with WIZ850IO (1)]()에 이어서, Device tree를 작성하는 내용부터 진행해 본다.

- ~~RPI zero 초기 설정~~
  - ~~이전 글 참조: [[RaspberryPi zero W] headless 초기 설정](https://renakim.github.io/2019/01/23/rpi-zero-w-start/)~~
- ~~RPI zero kernel compile~~
- **Device tree 작성 & 적용 (for W5500)**
- **동작 테스트**

---

### Device tree overlay

Device tree에 대한 상세 설명은 다음 링크를 참조한다.

- [라즈베리 파이 문서: 10. 디바이스 트리, 오버레이, 파라미터](https://wikidocs.net/3205)
- [Raspberry Pi doc: Device Trees, overlays, and parameters](ttps://www.raspberrypi.org/documentation/configuration/device-tree.md)

아래 내용은 RPI zero에서 직접 진행해도 무방하다.

---

---
layout: single
title: "[RaspberryPi] Hash Sum mismatch 에러"
categories:
  - Raspberry Pi
date: 2019-06-21
tags: [raspberrypi]
---

## Hash Sum mismatch 문제 발생

라즈베리파이를 사용할 일이 있어 최신 버전의 Rasbian을 설치했다. (2019-04-08-raspbian-stretch)

초기 설정 시 패키지 업데이트를 위해 apt-get update를 수행하는데, 자꾸 아래와 같은 에러가 발생하면서 정상 수행이 되지 않았다.

- 추가: apt-get update는 잘 수행되더라도 apt-get upgrade에서 문제가 발생할 수도 있다.

```
Hit:1 http://archive.raspberrypi.org/debian stretch InRelease
Get:2 http://mirrordirector.raspbian.org/raspbian stretch InRelease [15.0 kB]
Get:3 http://mirrordirector.raspbian.org/raspbian stretch/main armhf Packages [11.7 MB]
Err:3 http://mirrordirector.raspbian.org/raspbian stretch/main armhf Packages
  Hash Sum mismatch
  Hashes of expected file:
   - Filesize:11663116 [weak]
   - SHA256:dd920dbf4c20fb028a317ddd49b7c69b63d1e241e8bf51655f53f1c813772dc5
   - SHA1:6ddbe51cc9206a408cd479bd9a5c2e1e0ba98aee [weak]
   - MD5Sum:fe39fba2eddc33956d37b6a5c42f11ea [weak]
  Hashes of received file:
   - SHA256:44db7d86347ab0171b6e42506d07984c06ffcc57d3582a192ac08a989012b898
   - SHA1:cb6c1ab651449f437b7ec5d32fb469058c5c4db1 [weak]
   - MD5Sum:db0b7d2fd433fde31459c4d799b11bba [weak]
   - Filesize:11387934 [weak]
  Last modification reported: Wed, 19 Jun 2019 04:27:53 +0000
  Release file created at: Thu, 20 Jun 2019 22:29:14 +0000
...
...
```

Hash Sum mismatch 라는 문구가 눈에 띄어 검색해보니 내용이 많이 나오는데, 해결 방법은 바로 찾지 못해서 시간이 좀 걸렸다.

---

## 해결 방법

해결 방법은 Raspbian mirror 링크를 변경해 주는 것이다.

### Raspbian Mirror

먼저 아래 링크에 접속해서 변경할 새 mirror 링크를 찾는다.
링크는 지역 별로 있어서 적절히 가까운 지역으로 선택하면 되는 듯하다.
나는 South Korea 지역 링크 중 하나를 선택했다.

- [Raspbian Mirror List](http://www.raspbian.org/RaspbianMirrors)

---

### Mirror 링크 변경

라즈베리파이에 접속해서 mirror 링크를 새 링크로 변경해 준다.

설정 파일의 기존 라인을 복사한 다음 주석처리 하고, 링크 부분만 변경해 주었다.

```
http://raspbian.raspberrypi.org/raspbian/  ==>  http://mirror.premi.st/raspbian/raspbian/
```

```
$ sudo vi /etc/apt/sources.list

#deb http://raspbian.raspberrypi.org/raspbian/ stretch main contrib non-free rpi
deb http://mirror.premi.st/raspbian/raspbian/ stretch main contrib non-free rpi
# Uncomment line below then 'apt-get update' to enable 'apt-get source'
```

설정 저장 후 reboot.

다시 apt-get update 명령을 수행하니 잘 된다.

---

## Reference

- 참고 링크: [Raspberry Pi Forum: [Solved] PiVPN and Updates missing](https://www.raspberrypi.org/forums/viewtopic.php?f=66&t=242541&p=1479291&hilit=mismatch#p1479291)

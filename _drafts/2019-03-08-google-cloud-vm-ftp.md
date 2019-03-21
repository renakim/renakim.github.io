---
layout: posts
title: "[GCP] Compute Engine - VM 인스턴스 FTP 설정"
categories: Docker
date: 2019-03-08
tags: [gcp, vm, ftp, filezilla]
---

Google Cloud Platform(이하 GCP) 기반 Compute Engine 서비스에서 VM 인스턴스로 서버를 구축해 사용하고 있다.

이 때 파일 송수신 작업이 필요할 때가 많은데, 이를 위한 FTP 설정 방법을 정리해 본다.

## GCP: Google Compute engine

Google compute engine에 대한 설명은 아래를 참고한다. 한글로 설명이 잘 되어있다.

- [Google Compute engine 문서](https://cloud.google.com/compute/docs/)

---

## VM 인스턴스 생성

---

## PuTTYgen key 생성

PuTTygen을 이용해 public/private key를 생성한다.

### PuTTYgen

PuTTYgen은 PuTTY를 installer(윈도우의 경우 .msi)로 설치 시 함께 설치된다.

설치 파일은 아래 링크에서 다운로드 받을 수 있다.

- [Download PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

### Key 생성

초기 화면은 아래와 같이 생겼다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/puttygen-1.png?raw=true">

**Generate** 버튼을 눌러 key 생성을 시작한다.

Key part에 나와있는 설명을 보면 빈 부분에서 마우스를 움직이라고 되어 있는데, 실제로 마우스를 움직여줘야 progress bar가 진행된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/puttygen-2.png?raw=true">

Key 생성이 완료되면, 비어있던 칸에 random한 key가 생성된다.

이 때 **Key comment를 Google VM 인스턴스에서 사용하는 서버 ID로 지정**해 주도록 한다.

그 다음 **key 값을 복사**하고, **Save private key 버튼을 눌러 key를 파일로 저장**해 둔다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/puttygen-3_1.png?raw=true">

---

## VM 인스턴스 메타데이터 설정

이제 앞 단계에서 만든 Key를 클라우드에 적용해 준다.

[Google cloud console](https://console.cloud.google.com)로 접속해서 Compute Engine의 **메타 데이터** 서비스로 접속한다.

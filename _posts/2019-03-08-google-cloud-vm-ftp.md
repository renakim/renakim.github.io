---
layout: post
title: "[Google Cloud] Compute Engine - VM instance FTP 설정"
categories: Docker
date: 2019-03-08
tags: [gcp, vm instance, puttygen, ftp, filezilla]
background: "/files/gcp-metadata-2.png"
---

Google Cloud Platform(이하 GCP) 기반 Compute Engine 서비스에서 VM 인스턴스로 서버를 구축해 사용하고 있다.

이 때 파일 송수신 작업이 필요할 때가 많은데, 이를 위한 FTP 설정 방법을 정리해 본다.

## GCP: Google Compute engine

Google compute engine에 대한 설명은 아래를 참고한다. 한글로 설명이 잘 되어있다.

- [Google Compute engine 문서](https://cloud.google.com/compute/docs/)

추후 GCP를 이용해 프로젝트를 진행하면서 진행한 내용들을 정리해 볼 예정이다.

---

## PuTTYgen으로 key 생성

우선, PuTTygen을 이용해 public/private key를 생성한다.

### PuTTYgen

PuTTYgen은 PuTTY를 installer(윈도우의 경우 .msi)로 설치 시 함께 설치된다.

설치 파일은 아래 링크에서 다운로드 받을 수 있다.

- [Download PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

### Key 생성

실행 시 초기 화면은 아래와 같이 생겼다.

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

SSH 탭으로 이동해서 하단의 항목 추가 버튼을 누르고, 조금 전 복사한 key 값을 붙여넣는다.

이 때 key를 만들때 설정했던 comment 값이 왼쪽에 나타난다. 이는 유저 이름으로 사용된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/gcp-metadata-1.png?raw=true">

입력이 잘 되었으면 저장 버튼을 눌러 설정을 완료한다.

### Key 생성 확인

Key가 잘 생성되는지 확인하려면 VM 인스턴스에 SSH를 통해 접속해 보면 된다.

VM 인스턴스 서비스에 들어가서 해당되는 VM 인스턴스에 대해 **SSH** 버튼을 누르면 원격 접속이 가능하다.

로딩 중 아래와 같은 문구를 볼 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/gcp-metadata-2.png?raw=true">

이제 아래 명령을 입력해 key 파일 유무를 확인하고, cat 명령으로 내용을 확인해 볼 수 있다.

```
$ ls .ssh/authorized_keys

$ cat .ssh/authorized_keys
```

---

## Filezilla로 연결 (SFTP)

GCP 쪽 설정을 완료했으니 이제 PC에서 Filezilla client를 사용해 연결해 본다.

Filezilla를 실행하고 **편집-설정** 창을 실행한 뒤 **SFTP** 메뉴로 들어간다.

그리고 앞서 저장했던 Key를 Add key file 버튼으로 추가해 준다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/gcp-ftp-filezilla-1.png?raw=true">

설정을 저장하고 서버에 접속해 보자.

상단 메뉴 밑의 입력 창에 다음 정보를 입력하고 연결하면 된다. 비밀번호는 입력하지 않아도 된다.

- 호스트(Host): VM 인스턴스의 외부 IP
- 사용자명(User): Comment에 입력했던 username

### 사이트 설정

사이트 설정을 해두면 접속할 때마다 IP를 입력하지 않아도 된다.

파일 - 사이트 관리자(단축키: Ctrl + S)에 들어가서 새 사이트를 추가하자.

새 사이트 버튼을 누른 뒤, 아래 그림과 같이 설정해 주면 된다. 연결할 때 입력했던 정보를 동일하게 입력해주면 된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/gcp-ftp-filezilla-2.png?raw=true">

이제 사이트를 통해 쉽게 FTP에 접속할 수 있다.

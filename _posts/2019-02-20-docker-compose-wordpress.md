---
layout: single
title: "[Docker] Docker compose로 wordpress 설치"
categories: Docker
date: 2019-02-20
tags: [docker, docker-compose, wordpress, mysql]
---

참조: [Docker docs: Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)

Wordpress를 사용할 일이 생겨 설치법을 찾던 중 docker-compose를 사용하는 방법이 눈에 띄었다.

도커의 docker-compose 관련 공식 문서에서는 wordpress를 예시로 한 quickstart 문서를 제공하고 있다.
Wordpress 외에 Django, Rails 관련 문서도 있다.

Docker compose 관련 문서 목록은 아래 링크에서 확인할 수 있다.

- [Docker docs: docker compose](https://docs.docker.com/compose/)

## Docker

도커에 대한 기본 개념을 알고싶다면 아래 글을 읽어보는걸 추천한다.

- [초보를 위한 도커 안내서 - 도커란 무엇인가?](https://subicura.com/2017/01/19/docker-guide-for-beginners-1.html)

---

## Docker compose

Compose는 여러 개의 도커 컨테이너를 정의하고 실행할 수 있도록 해주는 Tool이다. YAML 파일 형식(.yml)을 사용해 어떤 컨테이너를 어떤 설정으로 실행할 것인지 정의한다.

자세한 설명을 아래 링크를 참조하자.

- [Overview of Docker Compose](https://docs.docker.com/compose/overview/)

개발 환경은 ubuntu 16.04 64bit로 VM을 사용했다.

### Install

Docker compose를 사용하려면 물론 Docker가 설치되어 있어야 한다.

설치가 되어있지 않다면 다음 명령으로 설치한다.

```
$ sudo apt-get install docker -y
```

다음, docker compose 설치 방법은 간단하다. 공식 문서에 개발환경 별로 잘 설명되어 있기 때문에 직접 가서 보는걸 추천한다.

- [Install Compose](https://docs.docker.com/compose/install/)

본 글에서는 ubuntu(linux) 환경에서 설치하는 방법을 요약하도록 한다.

#### Docker compose download

Github의 docker repository로부터 docker compose를 다운로드한다.

```
$ sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

#### 실행 파일 권한 설정

Binary 파일에 실행 권한을 추가한다.

```
$ sudo chmod +x /usr/local/bin/docker-compose
```

#### 설치 확인

설치가 잘 되었는지 확인하기 위해 버전 확인 명령을 준다.

```
$ docker-compose --version
docker-compose version 1.23.2, build 1110ad01
```

설치를 했는데 docker-compose 명령을 인식하지 못할 경우 심볼릭 링크를 추가한다.

```
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
```

---

## Wordpress 설치 (with docker-compose)

참조: [Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)

### docker-compose.yml 작성

docker-compose는 docker-compose.yml 이라는 설정 파일 내용에 따라 컨테이너를 설정하고 실행한다.

wordpress 설치를 위한 yml 파일을 다음과 같이 작성하자.

- dev로 설정된 부분들은 테스트를 위해 쉽게 설정한 것이다. 본인의 환경에 맞게 설정하면 된다.

```
$ vi docker-compose.yml
version: "3.3"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: dev
      MYSQL_DATABASE: dev
      MYSQL_USER: dev
      MYSQL_PASSWORD: dev

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: dev
      WORDPRESS_DB_PASSWORD: dev
      WORDPRESS_DB_NAME: dev
volumes:
  db_data: {}
```

### 프로젝트 빌드

docker-compose -up 명령으로 프로젝트를 시작한다.

```
$ docker-compose up -d
```

이제 **localhost:8000** 주소로 접속하면, 다음과 같이 wordpress 초기 설정을 진행할 수 있다.
VM 환경에서 진행했다면 localhost 대신 VM IP를 입력하면 된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/docker-wordpress-install_1.png?raw=true">

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/docker-wordpress-install_2.png?raw=true">

참고로 도커로 실행된 프로그램의 데이터는 volume이라는 컨테이너 데이터 저장소에 저장되며, 다음 글에서 volume 백업, 설정 등에 대한 내용을 다뤄 볼 예정이다.

---

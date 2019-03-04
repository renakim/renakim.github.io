---
layout: posts
title: "Docker compose로 wordpress 설치하기"
categories: Docker
date: 2019-02-20
tags: [docker, docker-compose, wordpress]
---

원글: [Docker docs: Quickstart: Compose and WordPress](https://docs.docker.com/compose/wordpress/)

## Docker

---

## Docker compose

https://docs.docker.com/compose/overview/

https://docs.docker.com/compose/install/

---

**docker-compose.yml**

```
version: "3.3"

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: devkim
      MYSQL_DATABASE: devkim
      MYSQL_USER: devkim
      MYSQL_PASSWORD: devkim

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: devkim
      WORDPRESS_DB_PASSWORD: devkim
      WORDPRESS_DB_NAME: devkim
volumes:
  db_data: {}
```

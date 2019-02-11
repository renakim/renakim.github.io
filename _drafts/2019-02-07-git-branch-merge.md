---
layout: posts
title: "[Git] Git branch/merge"
categories: Git
date: 2019-02-07
tags: [git, branch, merge]
---

---

## branch 생성 및 checkout

\$ git checkout -b iss53

위 명령은 아래 두 명령을 합친 것과 같다.

\$ git branch iss53

\$ git checkout iss53

## 수정 후 commit...완료 후 master로 전환

git add...git commit...
git checkout master

## Merge branch

git merge iss53

## branch 삭제

git branch -d iss53

---

### Reference

- https://git-scm.com/book/ko/v1/Git-%EB%B8%8C%EB%9E%9C%EC%B9%98-%EB%B8%8C%EB%9E%9C%EC%B9%98%EC%99%80-Merge%EC%9D%98-%EA%B8%B0%EC%B4%88

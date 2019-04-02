---
layout: post
title: "[Python] Version 설정 with Alternatives"
categories: Python
date: 2019-04-02
tags: [python, alternatives]
---

대부분의 Linux에는 python 2 & 3 버전이 모두 설치되어 있지만, 기본 설정된 버전은 2.x 인 경우가 많다.

이 때 python 명령에 대해 버전을 변경하는 방법을 정리해 본다. (아래 테스트는 raspbian에서 진행했다.)

설정된 pyhton 버전을 확인해보면, 다음과 같이 나온다.

```
$ python --version
Python 2.7.13
```

---

## Update-alternatives

update-alternatives는 리눅스에서 여러 버전의 패키지를 관리할 때 사용되는 도구이며, 심볼릭 링크를 관리해준다.

아직 python 패키지에 대한 정보가 없기 때문에 config을 하면 다음과 같은 error가 발생한다.

```
$ sudo update-alternatives --config python
update-alternatives: error: no alternatives for python
```

---

### Update-alternatives 등록

아래 명령으로 관리할 python package를 등록하자.

```
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python2.7 2
$ sudo update-alternatives --install /usr/bin/python python /usr/bin/python3.5 1
```

### Version config

등록을 완료했으면 이제부터 아래 명령으로 버전을 관리할 수 있다. 명령을 입력하고 사용하고자 하는 버전의 number를 입력하면 된다.

```
$ sudo update-alternatives --config python
There are 2 choices for the alternative python (providing /usr/bin/python).

  Selection    Path                Priority   Status
------------------------------------------------------------
* 0            /usr/bin/python2.7   2         auto mode
  1            /usr/bin/python2.7   2         manual mode
  2            /usr/bin/python3.5   1         manual mode

Press <enter> to keep the current choice[*], or type selection number: 2
update-alternatives: using /usr/bin/python3.5 to provide /usr/bin/python (python) in manual mode
```

이제 다시 python 버전을 확인해보면, 3.x 버전으로 변경된 것을 볼 수 있다.

```
$ python --version
Python 3.5.3
```

---

## .bashrc alias 등록

Config 명령이 너무 길다면, alias 등록을 통해 단축 명령어를 사용할 수 있다.

```
# .bashrc에 라인 추가
$ vi ~/.bashrc
alias chpy='sudo update-alternatives --config python'

# 변경사항 적용
$ source ~/.bashrc
```

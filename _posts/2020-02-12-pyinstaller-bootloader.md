---
layout: post
title: "[Pyinstaller] Bootloader compile"
categories: python
date: 2020-02-12
tags: [python, pyinstaller]
---

Python(pyqt5) 기반의 GUI Tool을 만들어서 사용할 때 pyinstaller로 생성한 exe 파일 다운로드 또는 실행 시 백신에 걸리는 문제가 종종 발생한다.

이 문제에 대한 해결책을 검색하던 중 pyinstaller bootloader를 환경에 맞게 컴파일하여 사용하는 방법이 있다고 해서 시도해 봤다.

이 방법을 시도하기 전에 인증서를 사용해 디지털 서명을 추가하는 방법을 시도해서 약간 효과를 봤었다. (예: 백신으로 인해 삭제되던 exe 파일이 서명 추가 후 삭제되지 않음)

이는 다음 글에서 정리하도록 한다.

Pyinstaller는 version 3.5를 사용하다가 얼마 전 3.6 version이 릴리즈 되어 업데이트 후 사용 중이다.

---

## 환경

- Windows 10 64bit
- Python 3.7.4

참고로 pip를 사용해서 pyinstaller를 설치한 경우, bootloader 이미지는 있으나 소스 코드는 포함되어 있지 않다고 한다.

pyinstaller 패키지가 설치된 경로로 들어가 보면 bootloader 디렉터리의 이미지들이 환경 별로 나누어져 있는 것을 볼 수 있다.

```
$ pwd
/c/Python37-32/Lib/site-packages/PyInstaller/bootloader

$ ls -al
total 12
drwxr-xr-x 1 Administrator 197121 0  2월 12 09:38 ./
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 ../
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 Darwin-64bit/
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 images/
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 Linux-32bit/
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 Linux-64bit/
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 Windows-32bit/
drwxr-xr-x 1 Administrator 197121 0  1월 29 11:20 Windows-64bit/

$ ls -al Windows-64bit/
total 1084
drwxr-xr-x 1 Administrator 197121      0  1월 29 11:20 ./
drwxr-xr-x 1 Administrator 197121      0  2월 12 09:38 ../
-rwxr-xr-x 1 Administrator 197121 274432  1월 29 11:20 run.exe*
-rwxr-xr-x 1 Administrator 197121 278016  1월 29 11:20 run_d.exe*
-rwxr-xr-x 1 Administrator 197121 273408  1월 29 11:20 runw.exe*
-rwxr-xr-x 1 Administrator 197121 278016  1월 29 11:20 runw_d.exe*
```

---

## Build and install

### Download source package

아래 링크에서 릴리즈 버전 별로 소스 코드를 다운로드 할 수 있다.
https://github.com/pyinstaller/pyinstaller/releases

또는, git clone을 사용하여 최신 개발 버전 또는 원하는 commit 버전을 받아올 수 있다.

```
git clone https://github.com/pyinstaller/pyinstaller
```

여기에서는 git clone을 이용해서 최신 커밋 버전을 다운로드 했다.

### Build

```
cd pyinstaller/bootloader
```

```
python ./waf distclean all
```

(Log)

```
'distclean' finished successfully (0.072s)
'all' finished successfully (0.001s)
'distclean' finished successfully (0.001s)
...
[21/21] Linking build\releasew\runw.exe
...

'install_releasew' finished successfully (0.148s)
```

### Install

```
cd ..
python setup.py install
```

(Log)

```
running install
running bdist_egg
running egg_info
writing PyInstaller.egg-info\PKG-INFO
writing dependency_links to PyInstaller.egg-info\dependency_links.txt
writing entry points to PyInstaller.egg-info\entry_points.txt
writing requirements to PyInstaller.egg-info\requires.txt
writing top-level names to PyInstaller.egg-info\top_level.txt
reading manifest file 'PyInstaller.egg-info\SOURCES.txt'
reading manifest template 'MANIFEST.in'
...
Using c:\python37-32\lib\site-packages
Searching for future==0.17.1
Best match: future 0.17.1
Adding future 0.17.1 to easy-install.pth file
Installing futurize-script.py script to C:\Python37-32\Scripts
Installing futurize.exe script to C:\Python37-32\Scripts
Installing futurize.exe.manifest script to C:\Python37-32\Scripts
Installing pasteurize-script.py script to C:\Python37-32\Scripts
Installing pasteurize.exe script to C:\Python37-32\Scripts
Installing pasteurize.exe.manifest script to C:\Python37-32\Scripts

Using c:\python37-32\lib\site-packages
Finished processing dependencies for PyInstaller==4.0.dev0+ga1f92c6a.mod
```

pyinstaller 디렉토리 내 setup.py 파일을 사용하여 설치 까지 완료했다.

---

## 동작 확인

### pyinstaller 버전 확인

직접 빌드 후 설지한 버전을 확인하니 아래와 같이 나온다.

```
$ pyinstaller -v
4.0.dev0+ga1f92c6a.mod
```

그리고 동일한 python 코드를 사용하여 exe 파일 생성 후, 확인해 봤는데 큰 차이는 알 수 없었다.

([Virus total 사이트](https://www.virustotal.com/){:target="\_blank"}에 exe 파일 업로드 후 결과를 확인해봤을 때 같은 결과가 나왔다.)

EXE 파일에 대한 테스트 방법에 대해서도 정리해 봐야겠다.

---

# 참조 링크

- [https://pythonhosted.org/PyInstaller/bootloader-building.html](https://pythonhosted.org/PyInstaller/bootloader-building.html){:target="\_blank"}
- [https://stackoverflow.com/questions/43777106/program-made-with-pyinstaller-now-seen-as-a-trojan-horse-by-avg](https://stackoverflow.com/questions/43777106/program-made-with-pyinstaller-now-seen-as-a-trojan-horse-by-avg){:target="\_blank"}
- [https://stackoverflow.com/questions/53584395/how-to-recompile-the-bootloader-of-pyinstaller](https://stackoverflow.com/questions/53584395/how-to-recompile-the-bootloader-of-pyinstaller){:target="\_blank"}

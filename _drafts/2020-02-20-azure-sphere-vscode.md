---
layout: post
title: "[Azure Sphere] Visual Studio Code 개발환경"
categories: azuresphere
date: 2020-02-20
tags: [azure, azuresphere, vscode]
comments: true
---

Azure sphere OS 19.11 버전부터 Visual Studio Code를 개발환경으로 사용할 수 있게 되었다.

평소 VSCode를

(참조: https://docs.microsoft.com/ko-kr/azure-sphere/resources/release-notes-1911#new-features-and-changes-in-the-1911-release)

# 개발환경 구성

- Windows 환경 설정 참조: https://docs.microsoft.com/ko-kr/azure-sphere/install/development-environment-windows#develop-with-vs-code

## Azure sphere SDK 설치

기존 개발환경으로 Visual Studio를 사용하고 있었기 때문에 Visual Studio용 Azure Sphere SDK Preview가 설치되어 있어서, 이를 그대로 사용했다.

## VS Code 및 확장 설치

Visual Studio Code가 설치되어 있지 않다면 설치하고, Azure Sphere 확장을 설치한다.
Azure Sphere 확장을 설치하면 아래 항목들이 추가된다.

- Azure Sphere
- C/C++
- CMake 도구

### VS Code CMake 설정

1. Visual Studio 설치 경로 확인

- 기본 설치 경로: `C:\Program Files (x86)\Microsoft Visual Studio\<year>\<sku>`

- 예) C:\Program Files (x86)\Microsoft Visual Studio\2019\Community

2. VSCode 설정 추가

Global 설정 추가: `VS Code에서 파일 > 기본 설정 > 설정 (단축키: Shift + ,)`

또는,

현재 프로젝트에만 설정: `.vscode\settings.json 파일에 추가`

```
"cmake.cmakePath": "<VS 설치 경로>\Common7\IDE\CommonExtensions\Microsoft\CMake\CMake\bin\cmake.exe"
"AzureSphere.ArmGnuPath": "<VS 설치 경로>\Linux\gcc_arm\bin"

# cmake.configureSettings 개체에서 Ninja의 위치 추가
"CMAKE_MAKE_PROGRAM": "<VS 설치 경로>\Common7\IDE\CommonExtensions\Microsoft\CMake\Ninja\ninja.exe"
```

- 역슬래시를 슬래시로 수정해 줌.

```
// settings for Azure sphere (cmake)
"cmake.cmakePath": "C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/CMake/bin/cmake.exe",
"AzureSphere.ArmGnuPath": "C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/Linux/gcc_arm/bin",
"cmake.configureSettings": {
    "CMAKE_MAKE_PROGRAM": "C:/Program Files (x86)/Microsoft Visual Studio/2019/Community/Common7/IDE/CommonExtensions/Microsoft/CMake/Ninja/ninja.exe",
}
```

### CMake 캐시 삭제 후 재구성

- VS Code에서 보기 > 명령 팔레트 선택
- CMake를 입력하고, 메뉴에서 [CMake: 캐시 삭제 및 다시 구성] 선택

---

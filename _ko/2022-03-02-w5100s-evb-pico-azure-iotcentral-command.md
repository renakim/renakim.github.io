---
layout: single
title: "Control W5100S-EVB-Pico with Azure IoT Central Commands"
categories: w5100s-evb-pico
date: 2022-03-02
tags: [W5100S-EVB-Pico, Azure, IoT Central, openssl, Raspberry Pi, RP2040, vscode, W5100S, WIZnet]
---

지난 글에서, W5100S-EVB-Pico를 X.509 인증서를 사용해 Azure IoT Central에 인증 및 연결하는 방법을 정리했었다.

- [[W5100S-EVB-Pico] Connect to Azure IoT Central with X.509 certificate](https://renakim.github.io/w5100s-evb-pico-azure-iotcentral/)


위 과정을 간략히 정리해본다면, 다음과 같이 나눌 수 있을 것 같다.

  1. W5100S-EVB-Pico 개발환경 설치
  2. IoT Central 어플리케이션 생성 및 설정
  3. X.509 인증서 생성 (Using OpenSSL)
  4. 프로젝트 다운로드 및 빌드
  5. 실행 및 모니터링


이번 글에서는 연결된 W5100-EVB-Pico에 C2D(Cloud to Device) 메시지를 전송하고, 이를 활용해 다른 장치를 제어하는 방법에 대해 정리해보고자 한다.

Azure IoT Central에서 C2D 메시지를 사용하는 방법은 몇 가지가 있는데 이 중 commands를 사용할 것이다.

Commands를 활용하면 원격지에서 장치를 제어하거나 설정을 변경하는 등 관리가 가능해진다. 또는, 장치로부터 상태 데이터를 수집할 수도 있다.

더 자세한 내용은 아래 가이드 문서에서 참조할 수 있다.

- [Azure IoT Central: How to use commands](https://docs.microsoft.com/en-us/azure/iot-central/core/howto-use-commands)


W5100S-EVB-Pico 디바이스를 기반으로, 관련 기능 사용을 위한 Azure IoT 함수를 추가하고 GPIO를 사용하여 외부 장치(Raspberry Pi 3)를 간단히 제어하는 내용을 정리해 본다.

**이전 프로젝트를 기반으로 하는 것이기 때문에, 1~3 Step은 선행되었다는 가정 하에 진행한다.**


# 장치 준비

이번 글에서 사용한 구성은 다음과 같다.

- [W5100S-EVB-Pico](https://docs.wiznet.io/Product/iEthernet/W5100S/w5100s-evb-pico)
- Raspberry Pi 3 (제어 대상, 다른 디바이스도 가능)
- Jumper cable
- Ethernet cable


기존 하드웨어 구성에 더해 제어 대상이 필요했는데, 책상에 놀고있던 라즈베리 파이가 하나 있어서 이를 사용했다. GPIO를 제어할 수 있고 python을 실행할 수 있다면 어떤 Pi를 사용해도 될 것 같다.

GPIO pin을 하나 선택해서 W5100S-EVB-Pico와 점퍼 케이블을 사용해 연결해 주었다.


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico_rpi3.jpg?raw=true" />



# 프로젝트 준비

지난번 글에서 Azure IoT Central에 연동하기 위한 예제로 프로비저닝 예제를 사용했었다.

다운받아둔 프로젝트가 있으면 그대로 사용하면 되고, 그렇지 않으면 git clone을 사용해 다운받는다.

```
git clone https://github.com/Wiznet/RP2040-HAT-AZURE-C
```

## 코드 수정 작성

### main.c

사용할 예제를 설정한다. X.509 Provisioning 예제를 사용했다.

```
// The application you wish to use should be uncommented
//
//#define APP_TELEMETRY
//#define APP_C2D
//#define APP_CLI_X509
#define APP_PROV_X509
```


### sample_certs.c

앞서 생성한 인증서를 변수의 값으로 추가한다.

| Variable               | Certificates           |
| ---------------------- | ---------------------- |
| pico\_az\_CERTIFICATE  | device.crt certificate |
| pico\_az\_PRIVATE\_KEY | device.key certificate |
| pico\_az\_id\_scope    | IoT Central ID scope   |
| pico\_az\_COMMON\_NAME | Device ID              |



### prov_dev_client_ll_sample.c

샘플 소스 코드에서 IoT Central Commands 호출에 대한 callback function을 추가하고 등록해준다.

관련 예제는 아래 링크에서 참조 가능하다.

- [azure-iot-sdk-c: device Twin and methods sample](https://github.com/Azure/azure-iot-sdk-c/blob/main/iothub_client/samples/iothub_client_device_twin_and_methods_sample/iothub_client_device_twin_and_methods_sample.c#L191)


아래 코드의 동작 내용은 다음과 같다.
- Commands 전송 시 Callback 함수 호출
- Method name 비교 (Name: powerReset 사용)
- 설정 동작 수행 및 응답 전송

W5100S-EVB-Pico에서는 GPIO15를 사용했다. (GP15)

Pinout은 다음 링크에서 참조 가능하다.
- [W5100S-EVB-Pico Pin-out](https://docs.wiznet.io/Product/iEthernet/W5100S/w5100s-evb-pico#pin-out)

옵션으로, 함수 동작 여부를 확인하기 위해 User LED 제어 부분을 임시로 추가했다.

```c
#define RESET_GPIO  15
#define LED_PIN     25
```

```c
static int deviceMethodCallback(const char* method_name, const unsigned char* payload, size_t size, unsigned char** response, size_t* response_size, void* userContextCallback)
{
    (void)userContextCallback;
    (void)payload;
    (void)size;

    int result;

    // User led on
    gpio_put(LED_PIN, 1);
    printf("Device method %s arrived...\n", method_name);

    if (strcmp("powerReset", method_name) == 0) {
        printf("\nReceived device powerReset request.\n");

        const char deviceMethodResponse[] = "{ \"Response\": \"powerReset OK\" }";
        *response_size = sizeof(deviceMethodResponse)-1;
        *response = malloc(*response_size);
        (void)memcpy(*response, deviceMethodResponse, *response_size);
        sleep_ms(500);

        gpio_put(RESET_GPIO, 1);
        sleep_ms(100);
        gpio_put(RESET_GPIO, 0);

        result = 200;
    } else {
        // All other entries are ignored.
        const char deviceMethodResponse[] = "{ }";
        *response_size = sizeof(deviceMethodResponse)-1;
        *response = malloc(*response_size);
        (void)memcpy(*response, deviceMethodResponse, *response_size);
        result = -1;
    }

    // User led off
    gpio_put(LED_PIN, 0);

    return result;
}
```

마지막으로 기존의 Device client handle에 device method callback을 등록한다.

```c
(void)IoTHubDeviceClient_LL_SetMessageCallback(device_ll_handle, receive_msg_callback, &iothub_info);
// added line
(void)IoTHubDeviceClient_LL_SetDeviceMethodCallback(device_ll_handle, deviceMethodCallback, NULL);
```



# 라즈베리파이 3 Python 코드 작성

제어 대상이 될 라즈베리 파이 3에서, GPIO 상태를 모니터링하고 그에 따른 동작을 수행할 Python 코드를 작성했다.

코드 작성은 Raspberry Pi에 접속해서 Vim 등의 편집기로 작성해도 되고, Host PC에서 작성 후 Samba나 scp로 코드를 복사해줘도 된다.

여기에서는 Samba 설정 후 코드를 VS Code에서 불러와 편집 및 테스트하면서 진행했다.

내용은 다음과 같다.

- BCM 모드 사용
  - Board 모드와 BCM 모드 중 선택 가능
  - GPIO Pin 기준인 BCM 모드 사용
  - Pinmap 참조: [Raspberry Pi Pinmap](https://pinout.xyz/)
- GPIO Falling event 감지
  - 이벤트 발생 시 설정한 동작 수행 (예: reboot)

`gpio_test.py`

```python
import RPi.GPIO as gpio
import time
import os

channel = 23

# Set numbering system
gpio.setmode(gpio.BCM)
# gpio.setmode(gpio.BOARD)

# BCM23 (pin 16)
gpio.setup(channel, gpio.IN, pull_up_down=gpio.PUD_UP)

def restart():
    print('System will be restart...')
    time.sleep(2)
    os.system("sudo reboot")

def on_falling(channel):
    print(f'gpio falling event detected! {channel}')
    restart()

try:
    # Add event detect and callback function
    gpio.add_event_detect(channel, gpio.FALLING, callback=on_falling, bouncetime=200)

    # Wait for event
    while True:
        time.sleep(1)
except KeyboardInterrupt:
    gpio.cleanup()
finally:
    gpio.cleanup()
```


# Azure IoT Central application 설정

## Device Template 작성

### Commands 항목 추가

기존 템플릿 구성에서, powerReset 이라는 이름으로 Command type capability를 추가했다.

Name 값은 실제 호출되는 method 이름이며 대소문자를 구분하므로 주의한다.


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico-iotcentral-commands_add.png?raw=true" />


저장 후, 배포까지 완료해야 변경사항이 적용된다.

>배포가 완료된 상태에서는 Publish 버튼이 비활성화 된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico-iotcentral-commands_publish.png?raw=true" />

이제 디비이스 상세 페이지로 가면 Commands 탭이 생성되어 있고 템플릿에서 추가했던 Command 항목이 있는 것을 확인할 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico-iotcentral-commands_tab.png?raw=true" />


이 탭을 통해 디바이스로 Commands를 전송할 수 있다.



# 빌드, 실행 및 모니터링


## W5100S-EVB-Pico app 빌드 및 업로드

수정한 코드를 빌드하고, 장치에 업로드한다.

빌드 및 업로드 방법은 이전 글에서 참조할 수 있다.

- [W5100S-EVB-Pico Build and Run](https://renakim.github.io/w5100s-evb-pico-azure-iotcentral/#build)

추가로, 실시간 동작을 모니터링 하기 위해 Tera Term을 사용했다.




## 라즈베리 파이 python 코드 실행

라즈베리 파이3 에 ssh로 접속 후 작성한 python 코드를 실행한다.

```
python gpio_test.py
```


## Commands 전송 및 동작 모니터링

디바이스의 Commands 탭으로 이동 후, 명령을 전송하기 위해 `Run` 버튼을 클릭한다.

명령이 연결된 디바이스로 전송되고, 응답을 수신하면 성공 명령으로 처리된다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico-iotcentral-commands_run.png?raw=true" />

Command History를 통해 수행 시간, 응답 등 명령 수행 로그를 확인할 수 있다.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico-iotcentral-commands_history.png?raw=true" />


디바이스 모니터링의 경우, 아래와 같이 구성하여 테스트하면서 진행했다.

좌측 터미널은 COM 포트를 통해 W5100S-EVB-Pico를 모니터링 하는 것이고 우측 터미널은 Raspberry Pi에 SSH로 접속한 화면이다. 

- IoT Central에서 Command 전송
- W5100-EVB-Pico에서 powerReset 커맨드 수신
- GPIO를 통해 RPI3으로 신호 전송
- RPI3에서 신호 수신하여 Reboot 수행

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico-iotcentral-commands_monitor.png?raw=true" />


커맨드가 정상적으로 수신되어 라즈베리 파이가 Reboot 되면서 SSH 연결이 끊긴 것을 확인할 수 있다.

이 글에서는 간단한 Reboot 동작만 적용했지만, 이를 응용하면 다양한 컴포넌트들을 제어하고 관리할 수 있을 것이다.

기회가 되면 그에 대한 내용들도 정리해 볼 예정이다.


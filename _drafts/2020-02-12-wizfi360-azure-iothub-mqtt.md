---
layout: post
title: "[MQTT] broker site and"
categories: mqtt
date: 2020-02-12
tags: [mqtt, mqttbroker]
---

# Azure IoT Hub에 MQTT Command를 사용해 연결하기

## MQTT 설정 및 연결

### MQTTSET

MQTT 연결 시 요구되는 값들을 설정합니다.

`AT+MQTTSET=<UserName>,<Password>,<ClientID>,<AliveTime>`

Azure ioT 각 값의 형식
* Username: {iothubhostname}/{device_id}/?api-version=2018-06-30
* Password: SAS 토큰
  * 형태: SharedAccessSignature sig={signature-string}&se={expiry}&sr={URL-encoded-resourceURI}
* ClientId: deviceId


### MQTTTOPIC

Publish/Subscribe Topic을 설정합니다.

`AT+MQTTTOPIC=<publish topic>,<subscribe topic1>`

>Azure IoT Hub에 MQTT로 연결하는 경우, Pub/Sub 토픽은 다음과 같은 규칙을 따릅니다.
>  * devices/WizFi360/messages/events/
>  * devices/WizFi360/messages/devicebound/#
>
>[MS Azure: 디바이스-클라우드 메시지 보내기](https://docs.microsoft.com/ko-kr/azure/iot-hub/iot-hub-mqtt-support#sending-device-to-cloud-messages)



### MQTTCON

MQTT Broker에 연결합니다.

`AT+MQTTCON=<enable_auth>,<broker IP>,<broker port>`



----



AT+CWMODE_CUR=1	
AT+CWDHCP_CUR=1,1	
AT+CWJAP_CUR="ssid","password"

AT+MQTTSET="{IoT Hub Hostname}/{Device Id}/?api-version=2018-06-30","{SAS Token}","{Device Id}",60
AT+MQTTTOPIC="devices/{Device Id}/messages/events/","devices/{Device Id}/messages/devicebound/#"
AT+MQTTCON=1,"{IoT Hub Hostname}",8883

SharedAccessSignature sr=WizFiTestHub.azure-devices.net%2Fdevices%2FWizFi360&sig=O7v9QrIR7u6jsFAkPPqo4%2F4smt3f1fXF7QYOnszkJVo%3D&se=1582107916





----

# Reference

* https://mntolia.com/10-free-public-private-mqtt-brokers-for-testing-prototyping/
* https://diy-project.tistory.com/26
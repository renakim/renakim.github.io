---
layout: post
title: "[Azure IoT Hub] MQTT.fx로 접속 및 통신 테스트"
categories: mqtt
date: 2020-02-19
tags: [azure, iothub, mqtt, mqttbroker]
---

MQTT.fx는 MQTT Clinet 프로그램이다.

https://mqttfx.jensd.de/index.php




----

# Azure IoT Hub에 MQTT Command를 사용해 연결하기

각 값의 형식
* Username: {iothubhostname}/{device_id}/?api-version=2018-06-30
* Password: SAS 토큰
  * 형태: SharedAccessSignature sig={signature-string}&se={expiry}&sr={URL-encoded-resourceURI}
* ClientId: deviceId


`AT+MQTTTOPIC=<publish topic>,<subscribe topic1>`

devices/WizFi360/messages/events/
devices/WizFi360/messages/devicebound/#


MQTTS 접속
test.mosquitto.org
8883



AT+CWMODE_CUR=1	
AT+CWDHCP_CUR=1,1	
AT+CWJAP_CUR="ssid","password"

AT+MQTTSET="{IoT Hub Hostname}/{Device Id}/?api-version=2018-06-30","{SAS Token}","{Device Id}",60
AT+MQTTTOPIC="devices/{Device Id}/messages/events/","devices/{Device Id}/messages/devicebound/#"
AT+MQTTCON=1,"{IoT Hub Hostname}",8883


SharedAccessSignature sr=WizFiTestHub.azure-devices.net%2Fdevices%2FWizFi360&sig=CwQ6tDOmAXoFC8VA%2FWc%2FqjuGi8IPlI7zmv5%2FlAu4sD4%3D&se=1582072062








----

# Reference

* https://mntolia.com/10-free-public-private-mqtt-brokers-for-testing-prototyping/
* https://diy-project.tistory.com/26
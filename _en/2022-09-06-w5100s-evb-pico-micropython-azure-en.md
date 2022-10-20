---
layout: single
title: "Connect to Azure IoT Hub using Micropython and MQTT on W5100S-EVB-Pico"
categories: w5100s-evb-pico
date: 2022-09-06
tags: [w5100s-evb-pico, micropython, azure, iot hub, rp2040, w5100s, wiznet]
---



The process of connecting W5100S-EVB-Pico to Azure IoT Hub by MQTT using Micropython and sending and receiving messages.

I used SAS Token authentication method for IoT Hub.

## Components

**H/W**

W5100S-EVB-Pico
Micro 5pin USB cable
LAN cable

**S/W**

* Thonny
  * RP2040 Micropython Development Environment
* [Azure IoT Explorer](https://github.com/Azure/azure-iot-explorer/releases)
  * Check device information
  * Telemetry monitoring
  * C2D transmission



# Prepare Azure Resource

## Create a Azure IoT Hub


There are various ways to create an Azure IoT Hub, such as the Azure portal, Azure CLI, REST API, etc. In the beginning, we mainly use the method of creating through the Azure portal.

Instructions can be found at the link below.

- [Create an IoT hub using the Azure portal](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-create-through-portal)

# Micropython Firmware

## build

The build operation used WSL2 (Ubuntu 20.04.4 LTS) environment.

```
rena@Rena-PC:~$ lsb_release -a
No LSB modules are available.
Distributor ID: Ubuntu
Description:    Ubuntu 20.04.4 LTS
Release:        20.04
Codename:       focal
```

>You can refer to the [official documentation of Micropython](https://docs.micropython.org/en/latest/develop/gettingstarted.html#compile-and-build-the-code) for information on building the build environment, such as installing tools .

The build process was referenced in the README within the Micropython Repository.

https://github.com/micropython/micropython/tree/master/ports/rp2

### Repository clone

Clone the Repository including the submodules and retrieve the submodules.

```
git clone https://github.com/micropython/micropython.git
cd micropython

git submodule update --init
```


### Build submodules

```
make -C ports/rp2 submodules
```

### mpy-cross build (MicroPython cross-compiler)

Before building the device firmware, the mpy-cross build must be preceded.

```
make -C mpy-cross
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-01.png?raw=true" />



### Build W5100S-EVB-Pico Device Firmware

W5100S_EVB_PICOBuild the firmware using any of the supported devices .

>A list of support can be found at [Micropython: ports/rp2/boards](https://github.com/micropython/micropython/tree/master/ports/rp2/boards).


```
cd ports/rp2
make BOARD=W5100S_EVB_PICO submodules
make BOARD=W5100S_EVB_PICO
```

This is the final build process. It takes at least a few minutes.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-02.png?raw=true" />


## Firmware Upload

Upload the firmware to the device.

### Enter Boot Mode

The shape of H/W v1.0 and v1.1 is slightly different, but the board I have is v1.0, so  supply power (USB cable) while pressing the BOOTSEL button of the board, it enters Boot mode.

In the case of v1.1, if you press the RUN button while holding down the BOOTSEL button, it enters the boot mode and there is no need to re-apply the power.

### Firmware Upload

The built firmware is located in the following path.

- `micropython/ports/rp2/build-W5100S_EVB_PICO`

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-03.png?raw=true" />


Upload the file `firmware.uf2`.

Now the work for firmware is done.

Next, install umqtt library, write some code with Thonny, and send/receive data to IoT Hub and monitor data using Azure IoT Explorer.

# Write device code

The example code was referenced in IoTMQTTSample code in Azure-Samples.

- ~~[Azure-Samples/IoTMQTTSample - src/MicroPython (master)](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/MicroPython)~~
- [Azure-Samples/IoTMQTTSample - src/MicroPython](https://github.com/Azure-Samples/IoTMQTTSample/tree/9f4f5b5ad36416ed1850dfb4d942d2b2dc9a626d/src/MicroPython)

**[2022.10.14] As the master branch was updated, the Micropython directory was removed. Edit with commit branch address**



## Get device information from IoT Explorer

You need to get the information to connect to Azure IoT Hub and write it in your code.

>See the [Azure IoT Hub Guide: Communicate with your IoT hub using the MQTT protocol](https://docs.microsoft.com/en-us/azure/iot-hub/iot-hub-mqtt-support#using-the-mqtt-protocol-directly-as-a-device) for what each field requires when communicating with MQTT

In the case of the example code, the data below should be obtained.

* Device Connection String
* Device SAS Token

>If you look at the code, it is implemented to parse the Connection String to obtain the Host name, Device Id, and Shared access key values.

There are several ways to get information, and among them, I used the IoT Explorer.


- [Azure IoT Hub Release](https://github.com/Azure/azure-iot-explorer/releases)

### Set up IoT Explorer IoT Hub connection

* Reference: [https://docs.microsoft.com/en-us/azure/iot-fundamentals/howto-use-iot-explorer#connect-to-your-hub](https://docs.microsoft.com/en-us/azure/iot-fundamentals/howto-use-iot-explorer#connect-to-your-hub)


First, need to grant access so that IoT Explorer can access IoT Hub.

Among the default permissions, `iothubowner` permission including all permissions will be granted to IoT Explorer.

Click `iothubonwer`, click the button to the right of the Primary connection string, copy the value, and then paste it into the window that appears when you click Add connection in IoT Explorer and save.


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-04_0.png?raw=true" />

If you set up this setting only once in the beginning, you can perform most of the tasks for IoT Hub and devices in the tool.

After creating the device, get the information value as shown in the figure.

**Device creation**

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-04.png?raw=true" />


**Copy Connection String**

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-05.png?raw=true" />


**SAS Token Creation and Copy**

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-06.png?raw=true" />


## Full code

I uploaded the device code to the link below.

- [https://github.com/renakim/W5100S-EVB-Pico-Micropython/tree/main/examples/Azure](https://github.com/renakim/W5100S-EVB-Pico-Micropython/tree/main/examples/Azure)


The following contents have been added and modified in the [original example code](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/MicroPython).

* W5100S Network Connection Settings
* Modification of telemetry message transmission
  * String -> Json string

Without changing to Json, it is difficult to identify the data in IoT Explorer.


## Install libraries

Install the library in the Thonny environment.

After selecting Tools - Manage packages from the top menu, enter `umqtt` and search.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-07.png?raw=true" />

Among the found packages, install umqtt.simple and umqtt.robust in order.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-08.png?raw=true" />

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-09.png?raw=true" />


If the installation is successful, when you click umqtt in the list on the left, the installed packages are displayed as follows.

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-10.png?raw=true" />


## Execution and monitoring

### Telemetry

```
connecting
Publishing
Sending message 0
Sending message 1
Sending message 2
Sending message 3
Sending message 4
Sending message 5
Sending message 6
Sending message 7
Sending message 8
Sending message 9
Sending message 10
waiting for message
Received message
b'message from IoT Hub'
```

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-11.png?raw=true" />


<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-12.png?raw=true" />


### C2D message

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-13.png?raw=true" />

<img src="https://github.com/renakim/renakim.github.io/blob/master/files/w5100s-evb-pico/pico-micropython-azure-14.png?raw=true" />




# Reference

* [RP2040-HAT-MicroPython Repository](https://github.com/Wiznet/RP2040-HAT-MicroPython)
* [Azure IoTMQTTSample - Micropython](https://github.com/Azure-Samples/IoTMQTTSample/tree/master/src/MicroPython)
* [Micropython forum post from nickehallgren](https://forum.micropython.org/viewtopic.php?t=12955&p=70605)


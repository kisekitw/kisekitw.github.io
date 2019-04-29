---
layout: post
title:  "Internet of Thing參考架構"
date:   2019-03-14
excerpt: "Internet of Thing參考架構"
tag:
- Internet of Things 
- IoT
- Architecture
comments: false
---

IoT參考架構包含硬體, 韌體(執行於硬體上的軟體), MQTT Broker, API Engine和用戶操作界面抽象模組, 利用此參考架構可輕易以實作將抽象模組取代，例如： 

![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080314/arch.png?raw=true "IoT Architecture")

在此實做架構中, 左側是智慧物件, 右側則是用戶設備, 兩者間所有通訊都透過雲端傳遞： 

* Device
智慧物件或智慧設備, 包含Sensors與Actuators或兩者, 可以是任何Micro Controller或Micro Processor, 例如Raspberry Pi。 

* Gateway
負責通訊傳輸, 例如Wi-Fi router。 

* MQTT Broker
負責建立智慧物件與雲端的通訊通道, 考量安全行時可採用MQTT over SSL或MQTTS。 

* API Engine
API Engine是一個Web伺服器應用程式, 有三個主要功能： 

1. 作為MQTT Client負責與MQTT Broker溝通 
2. 負責將資料寫入至儲存設備(例如MongoDB) 
3. 負責提供APIs給用戶端設備使用 

* MongoDB
MongoDB是一種文件式的NoSQL資料庫, 非常適合處理來自各設備的Sensor資料, 因為各種設備的資料結構都有所差異。 

* Web/Phone/Desktop App
提供簡單的用戶UI界面, 由於API Engine的機制, 我們可以使用任何可存取API的技術實作。 

資料的Sequence Diagram如下︰ 

![Alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080314/DataflowFromSensorToDB.svg?raw=true)
<img src="https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080314/DataflowFromSensorToDB.svg?raw=true">
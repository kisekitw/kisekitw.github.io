---
layout: post
title:  "HypriotOS Installation"
date:   2019-06-13
excerpt: "如何安裝並設置HypriotOS"
tag:
- Raspberry PI
- Docker 
- HypriotOS 
- Kubernetes 
comments: false
---

最近想測試將RPI3(Raspberry Pi 3)作為K8S的Worker Node，佈署應用程式的Docker容器上去跑跑。 
首先需要一個適合的作業系統，找到了一個叫做**HypriotOS**的作業系統，對Docker的支援非常完善且資料豐富。

## 燒錄作業系統需要準備下面材料
1. 映像檔
   可直接從官方網站下載︰   
    ```
    https://blog.hypriot.com/downloads/
    ```
    也可以透過terminal下載︰   

    ```
    curl -LO https://github.com/hypriot/flash/releases/download/2.3.0/flash
    chmod +x flash
    sudo mv flash /usr/local/bin/flash
    ```   
2. 讀卡機
3. 燒錄軟體
   推薦使用**flash tool**。

## Wifi組態設定
若想讓HypriotOS啟動時直接將Wifi設定好，則須再準備一份yml檔案，範本可從下面網址找到︰  
```
https://github.com/hypriot/flash/blob/master/sample/wifi-user-data.yml

```
修改write_files>content區塊，取代ssid、key_mgmt等資訊即可。  

## 執行燒錄
上面準備好相關檔案及工具後就可執行燒錄，請將相關檔案放到工作目錄下，並切換到該工作目錄資料夾輸入下面指令︰  
```
flash -u wifi.yml hypriotos-rpi-v1.10.0.img 
```




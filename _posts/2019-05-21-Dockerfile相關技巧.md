---
layout: post
title:  "Dockerfile異質平台架構製作Image"
date:   2019-05-21
excerpt: "在Ubuntu 1604製作Armhf的Docker Image"
tag:
- Dockerfile 
- Docker 
- Image 
- Armhf 
- Linux
comments: false
---

有玩過Docker的人大概都知道Linux上製作的Docker Image只能執行在Linxu作業平台，原因是Docker Container本來就基於LXC(Linux Container)技術，微軟一開始用Hyper-V跑Linux虛擬機器以執行Docker Container，之後才發展了自己的Docker for Windows，也就是Windows Container。這兩者對應的Image互不相容，這是因為底層分別基於Linux API與Windows API, ***架構差異影響很大***，這其實讓我覺得好像也不是這麽跨平台。 

在推廣以Docker為主軸作為服務提供時，我們必須製作Image的Base Image並放到Raspberry PI 3(RPI3)上執行，原本想說OS都是Linux分支，因此選擇在Linux開發程式並製作Base Image，沒想到拿到RPI3上竟然無法執行，原來RPI3是ARM架構... 

當然其中一個方法就是直接在RPI3上撰寫Dockerfile再編成Image，但做了幾次後感覺不太靠普。複製程式就算了，因為RPI效能的問題try&error編寫Dockerfile後重新編譯Image的時間實在太久，最後終於找到一個方法可以在Ubuntu編譯Arm架構的Image。

首先要安裝建立Image的工具:

```
apt-get install debootstrap
```

debootstrap是Debian提供用於建立系統的方案，會在你所指定的目錄下安裝一個基本的Debian系統，也可指定產生特定架構的系統，例如：

- amd64 
- arm64 
- armel 
- armhf 
- i386 

... 

接著可執行:

```
sudo debootstrap --arch armhf stretch stretch > /dev/null
```

完整的debootstrap的命令架構如下:

```
debootstrap --arch <ARCH> <VERSION> <DIRECTORY> <MIRROR URL>
```
* ARCH  
要安裝的CPU架構，例如arm64、armel等。 
* VERSION   
Debian的版本，詳細版本名稱請見[官方網站](https://www.debian.org/releases/index.zh-tw.html)。
* DIRECTORY   
要安裝的目錄。 
* MIRROR URL  
下載Debian套件的網址，也可設定內部網路的伺服器。 

下載好二進位檔案後，接著要將安裝目錄的內容壓縮成一個文件，再透過Pipe傳給docker import指令，末端指令是給予一個Image Tag以方便識別:
```
sudo tar -C xenial -c . | docker import - xenial - xenial/rpi:v1
```
以上就建立完成一個以xenial為基礎的armhf基底映像檔(Base Image)。 













---
layout: post
title:  "kubeadmin init發生拉取k8s.gcr.io映像檔失敗"
date:   2019-04-02
excerpt: "新版本Kubernetes在安裝佈署時會發生從k8s.gcr.io下載映像檔失敗"
tag:
- Kubernetes
- Docker 
comments: false
---

### 問題描述

新版K8S安裝佈署時需要從k8s.gcr.io下載必須的映像檔, 直接執行下列**初始化**指令通常會拉取映像檔失敗:

```
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
```
![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080402/error1.png?raw=true "kubeadm init error")

目前查到發生的原因有:
1. 國內防火牆問題
2. 預設就是不行(汗)

### 解決方法

docker.io有存放該映像檔, 可以通過下面指令拉取映像檔:

```
kubeadm config images pull
```
![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080402/pull_image.png?raw=true "pull image")

拉取映像檔後再次執行初始化指令就可以成功執行了!
![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080402/success1.png?raw=true "success init")
![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080402/success2.png?raw=true "success init")

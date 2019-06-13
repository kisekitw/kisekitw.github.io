---
layout: post
title:  "Kubernetes Installation"
date:   2019-06-13
excerpt: "如何在AWS EC2安裝並設置Kubernetes Master & Worker Nodes"
tag:
- Kubernetes 
- Docker 
- AWS
- EC2 
comments: false
---

利用AWS EC2測試安裝並設置下面K8S組態︰

**Role**|**HostName**|**OS**
:-----:|:-----:|:-----:
Master|k8s-master-1|ubuntu 18.04
Worker|k8s-node-1|ubuntu 18.04
Worker|k8s-node-2|ubuntu 18.04

## 建立EC2執行個體
1. 進到**EC2 Dashboard**從畫面中間點擊**Launch Instance**
   ![EC2 Dashboard Launch Instance](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080612/DashboardLaunchInstance.png?raw=true)
2. 
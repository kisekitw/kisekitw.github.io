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
1. 進入控制畫面
   進到**EC2 Dashboard**從畫面中間點擊**Launch Instance**       
   ![EC2 Dashboard Launch Instance](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/DashboardLaunchInstance.png?raw=true)

2. 選擇AMI(Amazon Machine Image)     
   作業系統選擇**Ubuntu Server 18.04 LTS(HVM)**。
   ![EC2 Amazon Machine Image](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/chooseAMI.png?raw=true)

3. 選擇執行個體類型     
   Master的最低規格至少要**4core CPU**、**16G RAM**；Worker可依據要執行的容器數量決定，但建議至少**2core CPU**、**4G RAM**。
   ![EC2 Amazon Machine Image](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/ChooseInstanceType.png?raw=true)

4. 設置安全群組
   依據需求設定該EC2執行個體對外的通訊埠(Port)。   
   預設會開啟SSH的22，若為Web Server則可能需開啟443(https)、80(http)。
   若想讓執行個體可互相Ping，則須開Type為**Custom ICMP**、Protocol為**Echo Request**的規則。
   
   ### 重要說三次
   **Master要讓Worker可加入叢集納管，必須開啟6443！！！**
   **Master要讓Worker可加入叢集納管，必須開啟6443！！！**
   **Master要讓Worker可加入叢集納管，必須開啟6443！！！**

   ![EC2 Security Group](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/securityGroup.png?raw=true)

5. Lanuch後會導頁至EC2 Dashboard
    ![EC2 Instance Dashboard](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/EC2Dashboard.png?raw=true)

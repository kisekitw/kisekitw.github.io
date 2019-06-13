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
   
   ## 很重要說三次   
   **Master要讓Worker可加入叢集納管，必須開啟6443！！！**   
   **Master要讓Worker可加入叢集納管，必須開啟6443！！！**   
   **Master要讓Worker可加入叢集納管，必須開啟6443！！！**   

   ![EC2 Security Group](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/securityGroup.png?raw=true)

5. Lanuch後會導頁至EC2 Dashboard
    ![EC2 Instance Dashboard](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/EC2Dashboard.png?raw=true)

經過上面步驟，就會建立好一個Master與兩個Worker，接著在Master安裝K8S運行環境。

## 安裝Docker CE
Docker的版本至少要1.9以上。    

1. 更新目前套件清單    
   ```
   sudo apt update
   ```
2. 安裝一些必須的套件包，可讓apt透過HTTPS使用套件   
   ```
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```
3. 加入Docker官方儲存庫的GPG key    
   ```
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   ```
4. 新增Docker儲存庫到APT   
   ```
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
   ```
5. 再次更新套件庫以加入Docker套件    
   ```
   sudo apt update
   ```
6. 安裝Docker    
   ```
   sudo apt install docker-ce
   ```
7. 安裝結束後會啟動daemon，可輸入下面指令確認服務是否正常運行    
   ```
   sudo systemctl status docker
   ```
   輸出結果︰     
       ![Master Docker Service Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/MasterDockerServiceStatus.png?raw=true)
8. 另外兩個Worker也須依照步驟安裝Docker

接下來開始安裝kubernetes環境。

## 安裝Kubernetes    
1. 安裝kubelet與kubeadm    

```
sudo su   

apt-get update && apt-get install -y apt-transport-https   

curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -        

cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF    

apt-get update && apt-get install -y kubeadm     

```   

2. 確認kubeadm版本
確認版本為1.8以上。    

```
kubeadm version
```   

3. 關閉swap   
   
   ```   
   swapoff -a   

   sed -e '/swap/ s/^#*/#/' -i /etc/fstab   

   free -m   

   ```   

4. 初始化Master
5. 設定kubectl組態
6. 開啟Shell Autocompletion(OPT)
7. 開啟Master也可佈署POD(OPT)
8. 安裝POD網路
9.  將Worker加入叢集
10. 確認叢集資訊
11. 確認節點資訊

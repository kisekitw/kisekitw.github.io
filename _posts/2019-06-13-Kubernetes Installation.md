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
   **pod-network-cidr**是POD使用的虛擬網路，會從該區間分配IP給POD。   
   ```   
   kubeadm init --pod-network-cidr=10.244.0.0/16   
   ```     
   後面參數是flannel的預設參數，可參考︰   
   ```
   https://github.com/coreos/flannel/blob/master/Documentation/kube-flannel.yml
   ```  

   此步驟成功後就會取得讓Worker加入的指令(內含Token)，於**步驟8**可執行此指令:   
   ```   
   kubeadm join 172.31.8.158:6443 --token 3galn6.5fmev255mxr0yrr1 \
    --discovery-token-ca-cert-hash sha256:a8199ad9df17dfff452075ed75e7267d484a3f2277732e7a97273b3904ef6b09
   ```   

5. 設定kubectl組態
   ```   
   mkdir -p $HOME/.kube/   
   sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config   
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```   

6. 開啟Shell Autocompletion(OPT)   
   ```   
   apt-get install -y bash-completion   
   echo "source /etc/bash_completion" >> ~/.bashrc   
   echo "source <(kubectl completion bash)" >> ~/.bashrc   
   source <(kubectl completion bash)
   ```   
7. 開啟Master也可佈署POD(OPT)   
   ```   
   kubectl taint nodes --all node-role.kubernetes.io/master-   
   ```   
8. 安裝POD網路
   以安裝**Flannel**為例，有興趣也可安裝**weave**。   
   ```   
   kubectl apply --namespace kube-system -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
   ```   
9.  將Worker加入叢集   
    Worker要執行**步驟1、2、3**後再執行下面指令加入Master。   
    ```  
    kubeadm join --token <token> <master-ip>:<master-port>
    ```  
    上面詳細資訊參考**步驟4** 。

10. 確認叢集資訊
    ```   
    kubectl cluster-info
    ```   
    指令輸出︰　　
    ![Cluster-info](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/cluster-info.png?raw=true)

11. 確認節點資訊   
    ```   
    kubectl get nodes
    ```  
    指令輸出︰　　
    ![Get Nodes Result](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080613/GetNodes.png?raw=true)


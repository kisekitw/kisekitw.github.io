---
layout: post
title:  "Kubernetes Networking"
date:   2019-06-21
excerpt: "介紹Node IP、Pod IP、Cluster IP、NodePort"
tag:
- Kubernetes 
- Docker 
- Network 
comments: false
---    
## NODE IP   
K8S叢集中的機器被稱為**Node**節點，可以是一台實體主機，也可以是一台虛擬機器，每個節點上應該都包含一組關鍵程序︰**kubelet、kube-proxy、Docker Engine**。   
Node IP就是每個節點實體網路卡的IP位址。

## POD IP
Pod是K8S中最小可被佈署的元件，因此一個Pod內的所有容器永遠都會在同一台機器上。每個Pod中有一個特殊容器︰**Pause**(gcr.io/google_containers/pause)，該Pod除了負責用以被監控整個Pod中容器的狀態外，還被用以簡化容器內外的通訊問題，透過Flannel、Weave等虛擬二層網路技術，會為每個Pod分配一個唯一的IP位址，就稱為**Pod IP**，可讓節點上的容器獲得**同屬於一個內網**且**不重複的**IP位址。   

![POD IP Network](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/PODIP.png?raw=true)  

在K8S的世界裡，一個Pod裡的容器與另外一台主機上的Pod容器能夠**直接**通訊。     

## CLUSTER IP    
Service創見時K8S會配置一個IP位址，即Cluster IP，此時kube-proxy發現一個新的服務後，就會將該IP跟通訊埠(Service Port)註冊至iptable規則中。當一個用戶存取該Service時，就會依據iptable讓kube-proxy隨機挑選一個後端的Pod來提供服務。   
Cluster IP有下面特性:   
1. Cluster IP只跟Service物件有關，由K8S管理並依Cluster IP位址區間分配IP。
2. 只能結合Service Port組合實際的通訊連接埠，單獨的Cluster IP不具備TCP/IP通訊基礎。
3. 存在於K8S叢集封閉空間中，**叢集外要存取則須做額外設定**。   

![CLUSTER IP Network](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/ClusterIP.png?raw=true)


## 開放Cluster IP給外界存取  
要開放Service的Cluster IP給外界存取需要擴充Service的定義︰將type＝NodePort，並給予一個通訊埠號，例如nodePort: 31002。它為需要對外存取的Service在該Node開啟一個對應的TCP通訊埠，接著只要透過Node IP + NodePort通訊埠號就可存取該服務。但這只解了Service部份的問題，Service通常伴隨負載平衡的需求，若叢集執行在公有雲上(例如GCP、AMAZON...)，可以直接將type=LoadBalancer，系統就會自動建立一個對應的Load balancer實例，並會取得一個給用戶端存取的IP位址；若是自己架設的K8S，則須自己準備Load Balancer，例如Nginx、Traefik等。








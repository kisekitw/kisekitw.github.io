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
## POD IP
Pod是K8S中最小可被佈署的元件，因此一個Pod內的所有容器永遠都會在同一台機器上。每個Pod中有一個特殊容器︰**Pause**(gcr.io/google_containers/pause)，該Pod除了負責用以被監控整個Pod中容器的狀態外，還被用以簡化容器內外的通訊問題，透過Flannel、Weave等虛擬二層網路技術，會為每個Pod分配一個唯一的IP位址，就稱為**Pod IP**，可讓節點上的容器獲得**同屬於一個內網**且**不重複的**IP位址。   

 ![POD IP Network](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/POD IP.png?raw=true)

 在K8S的世界裡，一個Pod裡的容器與另外一台主機上的Pod容器能夠**直接**通訊。     

 





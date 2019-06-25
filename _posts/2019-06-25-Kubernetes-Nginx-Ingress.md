---
layout: post
title:  "Kubernetes Nginx Ingress"
date:   2019-06-25
excerpt: "介紹開放K8S Service的Ingress服務"
tag:
- Kubernetes 
- Ingress  
- Ingress Controller 
- Nginx  
comments: false
---  
說到Ingress就必須先談Kubernetes中的Service資源物件，是最核心的物件之一，它主要解決Pod動態生成IP的存取問題。由於Pod的生命週期並不是永久的，掛掉後就不會再重新啟動，因此通常會藉由Replication Controller或**ReplicaSet**資源物件確保Pod可維持在Desired的數量。   

從下面結構可以了解Pod、Service和資源物件間的關係︰   
![Service Architecture](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080625/ServiceArchi.png?raw=true)   
Service定義了一個服務的存取入口位址，frontend的Pod透過這個入口位址存取Service背後一組Pod叢集實例，Service與後面的Pod叢集透過Label Selector關聯。   

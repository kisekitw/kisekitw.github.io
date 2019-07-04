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
## 概念介紹  

說到Ingress就必須先談Kubernetes中的Service資源物件，是最核心的物件之一，它主要解決Pod動態生成IP的存取問題，透過Service的標籤(Tag)選定一組Pod，Service資源物件就會自動監控/負載Pod，然後只需要對外開放Service IP即可，也就是**NodePort**模式。  

好了，那問題就在於Service**NodePort**的存取：   

![Service Architecture](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080625/ServiceArchi.png?raw=true)  

當有多個Service運行時，系統會對Node設定相對應的通訊埠號對應每個Service的通訊埠號，因此新增、刪除Service時對應的通訊埠號就須變動，這在GCP或AWS這種平台上會需要一直調整防火牆的Rules。一個初步的作法是在每個Node上透過反向代理伺服器(例如Nginx)監聽一個對外的通訊埠(例如80)，對叢集內則設定網域規則，當請求進來時就會轉發到對應的ClusterIP，統一了各Node上的監聽埠號，但每次服務實例有變動，還是免不了去更改反向代理伺服器的組態，當Node很多，這種工人智慧的作法太不靠普了，這時就需要**Ingress資源物件**的功能，它包含了：  

1. Ingress Controller 
2. Ingress︰需搭配Ingress Controller才能讓組態生效 
3. Nginx︰最新版的K8S已將其整合至Ingress Controller，不需要單獨佈署   

透過Ingress資源物件抽象化Nginx的組態設定，原先要一個一個修改Nginx組態，現在則是針對Ingress資源物件修改，而既然是資源物件，就可用yaml創建、更新。疑？那Ingress Controller有什麼用途?Ingress Controller會一直跟K8S API Server溝通，持續偵測叢集中Ingress的變化，若有變化就依據定義的模板生成一段Nginx組態配置，再更新到Nginx的Pod中。  

總而言之，Ingress是透過L7協定將眾多服務以單一IP提供存取最有用的方式，

## Service vs Ingress
Service彙整一組Pod，提供一層抽象的介面讓用戶不用在意Pod個別的IP，Ingress則是在Service上再提供一層抽象介面，讓系統可針對**Host**或**Path**執行路由，而路由的目標是多種服務，扮演像是**Smart Router**或叢集**Entrypoint**的角色。

## 實例佈署   

『ingress vs load balance????』

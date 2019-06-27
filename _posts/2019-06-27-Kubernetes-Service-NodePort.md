---
layout: post
title:  "Kubernetes Service NodePort介紹"
date:   2019-06-27
excerpt: "更詳細說明Service NodePort運作模式"
tag:
- Kubernetes 
- Service  
- NodePort 
- Load Balance  
comments: false
---  
## 概念

首先要先知道當一個Pod建立時，系統會依據內部網路機制分配一個IP，也就是所謂的**Pod IP**。假設這是一個Web應用程式的Pod，當然就必須能被其他實體(人或系統)存取。一種方式是SSH至Node內，然後再存取Pod IP︰   

![Access By SSH](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080627/accessbyssh.png?raw=true)   

如上圖使用者SSH至Node IP為18.218.68.23的主機後，在內部用curl存取http://10.244.2.30(Pod IP)。但這樣當然無法符合需求，真正想要的是從用戶自己的電腦而不用SSH認證就可瀏覽Web網站:   

![Access No SSH](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080627/accessnossh.png?raw=true)   

要想有這樣的效果，在Node中就要有一個機制能幫忙做請求(Request)對映(Mapping)並轉發，而這就是**Service**的功能。   

![Access By NodePort](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080627/accessbynodeport.png?raw=true)   

從上圖可看出Service的效果，其可在Node上開啟一個監聽的通訊埠號，稱為**NodePort**，外面的用戶就可對**Node IP + NodePort**發出請求，Service監聽到有請求後就負責轉發給其納管(符合selector)的Pods，至於會送往哪個Pod則完全是隨機的(Random)，這樣看起來Service很像是內建的Load Balancer。   

## 與組態檔的關係  
撰寫組態檔時，spec.ports[]可設定公開的通訊埠清單，分別有port、targetPort、nodePort︰   

![Simple Service Config](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080627/simpleserviceconfig.png?raw=true)   

對應到示意圖:   

![Simple Service](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080627/simpleservice.png?raw=true)   

1. TargetPort 
   要轉發至Pod的通訊埠號，非必填，預設與Port相同。
2. Port 
   服務監聽的通訊埠號，**必填**。
3. NodePort 
   當spec.type=NodePort時，指定Node開放的通訊埠號，非必填，若沒有填則會自動分配**30000-32767**範圍的值。  

上面設定提供了所有讓請求進來的叢集的相關資訊。  

## 更複雜的範例  
上面範例都假定單一Node與單一Pod，但更為實際的範例通常為了實作High Availability及Load Balancing，會配置多個Nodes與多個Pods:   

![Complex Service](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080627/complexservice.png?raw=true)   



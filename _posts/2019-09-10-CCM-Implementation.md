---
layout: post
title:  "Kubernetes Cloud Controller Manager(CCM)功能實作"
date:   2019-09-10
excerpt: "介紹Kubernetes Cloud Controller Manager與OpenStack的實作方式"
tag:
- Kubernetes 
- Cloud Controller Manager
- OpenStack
comments: false
---  

### 前言    

先前談到KCM(Kubernetes Controller Manager)中與雲端相關的元件及功能描述如下:    

1. 節點控制器(Node Controller)   
    * **Initialize a node** with cloud specific zone/region labels
    * **Initialize a node** with cloud specific instance details, for example, type and size
    * Obtain the node’s network addresses and hostname
    * In case a node becomes unresponsive, check the cloud to see if the node has been deleted from the cloud. If the node has been deleted from the cloud, delete the Kubernetes Node object
2. 路由控制器(Route Controller)   
    * Responsible for configuring routes in the cloud appropriately so that containers on different nodes in the Kubernetes cluster can communicate with each other  
    * **Only** applicable for Google Compute Engine clusters
3. 服務控制器(Service Controller)   
    * Responsible for listening to service create, update, and delete events  
    * Configures **cloud load balancers** to reflect the state of the services in Kubernetes
4. 卷控制器(Volume Controller)   

Cloud Controller Manager(CCM)包含前三項，最後的Volume Controller因其涉及的複雜性與現有供應商獨特卷的實作邏輯而獨立於CCM外。  

為了要實作相關功能，供應商實作前必須先載入相依套件，即K8S提供的**k8s.io/cloud-provide**，github網址如下:   

```
https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/cloud-provider   
```
關於介面的制定都由**Sig-cloudprovider**維護。

### 以節點控制器實作為例子   
OpenStack提供的CCM實作都放在**pkg/cloudprovider/providers/openstack**下:   

![Openstack CCM Provider Folder](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080910/OpenstackCCMFolder.png?raw=true)  

### Gophercloud SDK

### Golang的介面實作

### Customer Controller的可能性



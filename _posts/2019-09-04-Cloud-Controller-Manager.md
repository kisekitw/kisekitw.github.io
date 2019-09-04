---
layout: post
title:  "Kubernetes Cloud Controller Manager(CCM)整合方法"
date:   2019-09-04
excerpt: "介紹Kubernetes Cloud Controller Manager與OpenStack整合方式"
tag:
- Kubernetes 
- Cloud Controller Manager
- OpenStack
comments: false
---  
### 從Cloud Provider到Cloud Controller Manager

1. In Tree ?
K8S 1.4版前的原始碼包含很多In Tree的Cloud Provider，當下載K8S後預設可透過配置直接使用這些Cloud Provider:

* OpenStack
* AWS
* AZURE
* GCE
* vSphere
* ...

 ![K8S 1.4 Cloud Provider](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080904/k8s14CloudProvider.png?raw=true)

```
https://github.com/kubernetes/kubernetes/tree/release-1.14/pkg/cloudprovider/providers
```

2. In Tree模式的問題

* K8S的核心能力應專注為一個指揮調度的工具，這些不同平台的Provider應由各供應商自行維護
* 對於特定的佈署情況，不需要下載用不到的Provider
* 這些Provider的重大更新耦合K8S更新周期
* 某些Provier已暫停更新

### 

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

1. In Tree? Out of Tree?  
K8S 1.4版前的原始碼包含很多In Tree的Cloud Provider，當下載K8S後預設可透過配置直接使用這些Cloud Provider:

* OpenStack
* AWS
* AZURE
* GCE
* vSphere
* ...

 ![K8S 1.4 Cloud Provider](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080904/k8s14CloudProvider.JPG?raw=true)


2. 

### 

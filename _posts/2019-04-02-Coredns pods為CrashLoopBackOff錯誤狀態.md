---
layout: post
title:  "Coredns pods為CrashLoopBackOff錯誤狀態"
date:   2019-04-02
excerpt: "檢查K8S Master節點時, Coredns pods呈現CrashLoopBackOff狀態"
tag:
- Kubernetes
- Docker 
comments: false
---

### 問題描述

初始化Master節點後, 執行下面指令檢查各服務狀態:

```
kubectl get pods --all-namespaces
```

![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080402/pods_error_status.png?raw=true "pods error status")

coredns呈現**CrashLoopBackOff**狀態。

### 解決方法

1. 修改CoreDNS configmap
```
kubectl -n kube-system edit configmap coredns 
```
2. 移除或註解**loop**
3. 移除CoreDNS Pods
```
kubectl -n kube-system delete pod -l k8s-app=kube-dns 
```

ref: https://www.reddit.com/r/kubernetes/comments/9fl7cg/coredns_crashloopbackoff_on_fresh_install/



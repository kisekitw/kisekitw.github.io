---
layout: post
title:  "Kubernetes Web Dashboard Installation"
date:   2019-06-20
excerpt: "介紹如何安裝K8S Web Dashboard"
tag:
- Kubernetes 
- Dashboard  
comments: false
---    

安裝好K8S叢集後，還需要一個GUI介面讓管理者查看目前叢集的狀態，下面說明如何安裝官方提供的Dashboard GUI。   

## 佈署Dashboard   
首先要透過kubernetes-dashboard.yaml佈署Dashboard︰    
```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```
上面指令會幫我們建立相關的K8S資源，接著可以輸入下面指令確認是否有成功執行︰
```
$ kubectl get deployment -n kube-system | grep dashboard
```  
結果如下顯示Dashboard的Deployment則表示成功︰
![Dashboard Deployment Check](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/dashboardStatusCheck.png?raw=true)    

## 修改Service為NodePort類型
接著要將Dashboard服務開放給叢集外部用戶存取，因此須修改type為**NodePort**並給予一個通訊埠號︰
``` yaml
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard","namespace":"kube-system"},"spec":{"ports":[{"port":443,"targetPort":8443}],"selector":{"k8s-app":"kubernetes-dashboard"}}}
  creationTimestamp: "2019-06-20T04:53:32Z"
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
  resourceVersion: "177698"
  selfLink: /api/v1/namespaces/kube-system/services/kubernetes-dashboard
  uid: 5a7b2ddc-9317-11e9-88b6-0292f50b6596
spec:
  clusterIP: 10.108.78.240
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32149   <------ modify
    port: 442
    protocol: TCP
    targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
  sessionAffinity: None
  type: NodePort      <------ modify
status:
  loadBalancer: {}

```



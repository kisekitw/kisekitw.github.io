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
接著要將Dashboard服務開放給叢集外部用戶存取，因此須修改type為**NodePort**並給予一個通訊埠號(必須在**30000-32767**)︰   

```shell
$ kubectl edit svc kubernetes-dashboard --namespace=kube-system
```

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
修改好後會自動套用新的組態。   

## 建立Service Account(RBAC)  
首先建立一個yaml檔案，命名為k8s-dashboard-adminuser.yaml(可隨意命名)並編輯該檔案:
```
$ sudo touch k8s-dashboard-adminuser.yaml   
$ sudo nano k8s-dashboard-adminuser.yaml
```   
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kube-system
```
除存檔案後就可執行下面指令apply該組態：   
```
$ kubectl apply -f k8s-dashboard-adminuser.yml
```

## 建立ClusterRoleBinding(RBAC)   
接著建立ClusterRoleBinding，將上面建立的角色繫結至叢集。   
建立一yml檔案，命名為clusterRoleBinding.yaml(可隨意命名)並編輯該檔案：
```
$ sudo touch clusterRoleBinding.yaml   
$ sudo nano clusterRoleBinding.yaml   
```
```yaml   
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kube-system
```   
除存檔案後就可執行下面指令create該組態：   
```
$ kubectl create -f clusterRoleBinding.yaml
```  
最後可執行下面指令檢查是否有成功繫結:
```
$ kubectl describe ClusterRoleBinding/admin-user
```   
結果應如下圖：
![Cluster Role Binding](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/clusterRoleBinding.png?raw=true)   

## 存取Dashboard   
要存取Dashboard的Service要先取得NodePort:   
```
$ kubectl describe svc kubernetes-dashboard --namespace=kube-system
```
```yaml
Name:                     kubernetes-dashboard
Namespace:                kube-system
Labels:                   k8s-app=kubernetes-dashboard
Annotations:              kubectl.kubernetes.io/last-applied-configuration:
                            {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"k8s-app":"kubernetes-dashboard"},"name":"kubernetes-dashboard"...
Selector:                 k8s-app=kubernetes-dashboard
Type:                     NodePort
IP:                       10.108.78.240
Port:                     <unset>  442/TCP
TargetPort:               8443/TCP
NodePort:                 <unset>  32149/TCP
Endpoints:                10.244.2.24:8443
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```   
接著用Node IP + NodePort就可以存取Dashboard：   
![Dashboard Login UI](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/DashboardUI.png?raw=true)    

Kubernetes儀表板提供兩種登入的方式，由於上面步驟有建立一個Service Account，系統會自動建立一個**secret**資源，因此可以用裡面的token作為與API Server認證使用。   
可使用下面指令取得token︰   
```
$ kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')
```
![Cluster Role Binding](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/token.png?raw=true)    
取得token後就可輸入登入畫面:   
![Input Token](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/InputToken.png?raw=true)     
![LoginSuccess](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080621/LoginSuccess.png?raw=true)    






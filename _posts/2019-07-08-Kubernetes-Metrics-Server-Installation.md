---
layout: post
title:  "Kubernetes Metrics Server Installation"
date:   2019-07-08
excerpt: "安裝K8S Metrics Server"
tag:
- Kubernetes 
- Metrics Server    
comments: false
---  

### 介紹   

Kubernetes v1.1開始新增了Horizontal Pod Autoscaler(HPA)控制器，該控制器會依據--horizontal-pod-autoscaler-sync-period定義的時間(預設為15秒)週期性監控Pod的**CPU使用率**(或一些**自定義的指標**)而自動擴展/縮容Pod(支持 replication controller、deployment 和 replica set))，當中Pod CPU的使用率來自~~Heapster元件，所以需要預先安裝好Heapster~~(Kubernetes v1.8被取代)Metrics-Server，所以需要預先安裝好Metrics Server(Kubernetes v1.11後預設使用)。  

### 安裝  

1. Clone程式碼
    ```
    https://github.com/kubernetes-incubator/metrics-server
    ```
2. 安裝  
    進到metrics-server資料夾，執行指令建立資源物件:   
    ```
    # Kubernetes > 1.8
    $ kubectl create -f deploy/1.8+/
    ```
3. 測試  
    ```
    $ kubectl top node
    error: metrics not available yet
    
    $ kubectl top pod
    W0708 07:40:22.141359   30041 top_pod.go:259] Metrics not available for pod default/job-wq-2-74bsz, age: 119h5m31.141350957s
    error: Metrics not available for pod default/job-wq-2-74bsz, age: 119h5m31.141350957s

    ``` 
    系統有可能需要收集一段時間才會正常顯示資訊，但等了很久還是一樣的狀態，可查看logs是否有異常。  
4. 查看Logs 
    ```
    $ kubectl logs -n kube-system [matrics-server-pod-name]
    ``` 
    結果如下:

    ![Matrics Server Error](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080708/matrics-server-error.png?raw=true)   

    從上面訊息發現是**x509: certificate signed by unknown authority**的問題，透過kubeadm佈署的叢集會出現該錯誤，須修改metrics-server的deployment檔案。 
5. 修改檔案
    ```
    kubectl edit deploy -n kube-system metrics-server
    ```
    修改前:   
    ```
    spec:
        containers:
        - image: k8s.gcr.io/metrics-server-amd64:v0.3.3
          imagePullPolicy: Always
    ```
    修改後:    
    ```
    spec:
        containers:
        - image: k8s.gcr.io/metrics-server-amd64:v0.3.3
          command:
          - /metrics-server
          - --kubelet-insecure-tls
          imagePullPolicy: Always
    ```
    除存修改後系統就會自動重新建立一個metrics-server的Pod。  
6. 查看Logs 
    ```
    $ kubectl logs -n kube-system [matrics-server-pod-name]
    ``` 
    結果如下表示成功：
    ```
    I0708 08:09:46.917336       1 serving.go:312] Generated self-signed cert (apiserver.local.config/certificates/apiserver.crt, apiserver.local.config/certificates/apiserver.key)
    I0708 08:09:47.208117       1 secure_serving.go:116] Serving securely on [::]:443  

    ...

    ```
7. 查看Metrics
    ```
    $ kubectl top nodes
    ```
    結果如下:
    ```
    NAME               CPU(cores)   CPU%   MEMORY(bytes)   MEMORY%   
    ip-172-31-0-138    41m          2%     1119Mi          29%       
    ip-172-31-40-242   22m          2%     519Mi           58%       
    ip-172-31-8-158    134m         3%     1667Mi          10%  
    ```



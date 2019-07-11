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

### 設置Proxy Server  
有了Metrics Server & Metrics API後，接下來就是要能透過curl、wget、或http客戶端(程式或服務)存取，因此要有一個反向代理伺服器來拋送請求。  

目前要存取API Server官方有提供兩種作法，可參考下面網址：   
```
https://k8smeetup.github.io/docs/tasks/access-application-cluster/access-cluster/
```
這邊就採用推薦的方式: **以proxy模式運行kubectl**，首先要將其作為系統服務以長時間在背景運行，步驟如下:   

1. 首先在/lib/systemd/system/建立一個空的kubectlproxy.service檔案並將下面內容複製到裡面
    ```
    [Unit]
    Description=kubectl proxy 8088
    After=network.target

    [Service]
    User=root
    ExecStart=/bin/bash -c "/usr/bin/kubectl proxy --address='0.0.0.0' --port=8088 --accept-hosts='^*$'"
    StartLimitInterval=0
    RestartSec=10
    Restart=always

    [Install]
    WantedBy=multi-user.target
    ```
2. 確認服務狀態
    ```
    systemctl -l status kubectlproxy.service
    ```
    如果狀態如下，則表示服務未正常運行：
    ```
    kubectl-proxy.service - kubectl proxy 8088
    Loaded: loaded (/lib/systemd/system/kubectl-proxy.service; disabled; vendor preset: enabled)
    Active: inactive (dead)
    ```   
3. 重新啟動
    ```
    systemctl daemon-reload   
    systemctl reenable kubectlproxy.service   
    systemctl restart kubectlproxy.service
    ```   
### 測試Web API Server  
開好伺服器後就可以用Postman測試是否可以擷取到叢集相關資訊。  
Metrics Server會從K8S集群中每個Node上kubelet API收集Metrics，接著透過Metrics API就可取得K8S物件资源的Metrics指標，Metrics API掛載在/apis/metrics.k8s.io/下。   

![Matrics Server API](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080708/MetricsServerWebAPI.png?raw=true)    

從上面可知道目前Metrics API提供的資源物件指標有**Node**和**Pod**，並都提供**get**、**list**的操作，可以進一步查詢**目前**叢集中Pod的狀態：   

![Matrics Server API Pod](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080708/PodStatus.png?raw=true)    

### 測試UI實做   
實做UI介面存取Metrics API： 

![K8S Pods Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080708/K8SPodStatus.png?raw=true) 

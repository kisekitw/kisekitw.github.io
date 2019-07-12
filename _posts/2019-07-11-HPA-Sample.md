---
layout: post
title:  "HPA - Use Case Sample"
date:   2019-07-11
excerpt: "官方HPA範例"
tag:
- Kubernetes 
- Metrics Server    
- Metrics API   
comments: false
---  

### 介紹   

Kubernetes官方介紹HPA功能時啟動一個php-apache的RC個體，接著建立HPA實體來控制Pod數量，這邊有個很重要的點是:**Deployment/RC資源物件必須先存在**且**Pod必須定義resources.requests.cpu**。範例接著會再起一個用戶端實例對php-apache作壓力測試，觀察Pod的增減情況。

### 說明   

在RC中會定義Pod的requests.cpu設定容器需要的最小資源，範例中設定為200m，表示該容器在一般情況下需要50%的CPU，在schedule時會保證所有Pod的requests總和小於Node能提供的計算能力；當CPU資源充足時，requests.cpu不會限制容器可用的最大值。   

### 範例測試

1. 執行並開放PHP-Apache Server
    ```
    kubectl run php-apache --image=gcr.io/google_containers/hpa-example --requests=cpu=200m --expose --port=80
    ```
2. 建立HPA  
    ```
    kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10    

    kubectl get hpa   
    ```   

    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/hpa_init.png?raw=true)   


3. 負載測試  
    開啟一個busybox容器並進到sh終端機中：
    ``` 
    kubectl run -i --rm --tty load-generator --image=busybox /bin/sh
    ```
    接著輸入下面指令增加請求負載:
    ```
    while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done
    ```



4. 觀察結果
    ```
    kubectl get hpa   

    kubectl get deployment php-apache   

    ```
    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/test1.png?raw=true)   

    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/test2.png?raw=true)   

    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/test3.png?raw=true)   

    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/test4.png?raw=true)   

    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/test5.png?raw=true)   

    ![HPA Initial Status](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080711/test6.png?raw=true)   

---
layout: post
title:  "Docker Container Execution Time Measure"
date:   2019-07-23
excerpt: "測量Docker容器啟動時間的方法"
tag:
- Docker 
- Container
- Time Measure
comments: false
---  

1. 建立Dockerfile，塞入大量資料
    ```
    FROM debian:stretch

    RUN dd if=/dev/urandom of=/largefile bs=1024 count=524288
    ```
2. 建制映像檔
    ```
    $ docker build -t kisekitw/largeapp .
    ```   
    
    ![Matrics Server Error](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080723/largeImageSize.png?raw=true)   

3. 執行容器
    ```
    $ sudo docker run --name=test kisekitw/largeapp ping -c 10 8.8.8.8
    ```
4. 計算啟動時間
    ``` bash
    START=$(sudo docker inspect --format='{{.Created}}' test)
    STOP=$(sudo docker inspect --format='{{.State.StartedAt}}' test)
    START_TIMESTAMP=$(date --date=$START +%s)
    STOP_TIMESTAMP=$(date --date=$STOP +%s)
    echo $(($STOP_TIMESTAMP-$START_TIMESTAMP)) seconds
    ```   
5. 結果

    ![Matrics Server Error](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080723/startupsecond.png?raw=true)   

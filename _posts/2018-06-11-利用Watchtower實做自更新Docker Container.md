---
layout: post
title:  "利用Watchtower實做自更新Docker Container"
date:   2018-06-11
excerpt: "利用Watchtower實做自更新Docker Container"
tag:
- Watchtower 
- Docker 
- Image 
- Self-Updating 
comments: false
---

[![Docker Container Self-Updating](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080522/self_updating_docker.png?raw=true)](https://www.slideshare.net/kisekitw/self-updating-service-on-rpi-3-using-docker)

Watchtower是一個應用程式，用以監控執行中的Docker containers的Image檔是否有變更。若有變更，則會自動用新的映像檔重新啟動容器(Gracefully)。 

該應用程式可以偵測Docker Hub或私有的Image Registry，它機制上是用定時輪詢的方式監控，因此可以為它設定時間(例如30秒)。

管理上完全自動化的更新似乎不是很好，因該要配合一個驗證關卡，確定更新的Image沒問題。 










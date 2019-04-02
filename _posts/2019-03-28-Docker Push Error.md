---
layout: post
title:  "Docker push錯誤"
date:   2019-03-28
excerpt: "上傳映像檔時出現denied: requested access to the resource is denied"
tag:
- Docker 
- Container 
comments: false
---

# 執行Docker push指令出現**denied: requested access to the resource is denied**錯誤 

![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080328/docker_push_error.png?raw=true "Error message")

上傳映像檔時必須先執行登入指令： 
```
docker login
```

輸入帳號密碼後就可執行上傳指令： 
```
docker push YOUR_DOCKERHUB_NAME/firstimage
```

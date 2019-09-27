---
layout: post
title:  "Cloud-init Inject User Data"
date:   2019-09-27
excerpt: "介紹如何在Flask中處理CORS"
tag:
- OpenStack 
- Cloud Init
comments: false
---  
### 創建OpenStack VM執行個體   

要創建的VM執行個體時，組態檔需注入帳密，之後才可登入VM(也可用keypair方式):      

![Create VM Metadata](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/userdata.PNG?raw=true)     

+ 繫結Floating-ip --> VM執行個體的狀態必須為**ACTIVE**

### Cloud-Init流程    

1. VM執行個體啟動時會透過**Cloud-Init**功能抓取初始化相關資料(ex:ssh-key、password、config...))
    VM的映像檔必須要有此功能。
2. 透過DHCP的設定VM會知道去存取http://169.254.169.254
3. 透過Router(192.168.130.1、192.168.130.2)以unix socket傳送訊息到HAProxy
4. HAProxy傳送訊息到Neutron Metadata Proxy(Metadata Server?)
5. Neutron Metadata Proxy再存取Nova API取得相關資訊(User-data.txt)回傳給VM
6. VM執行User-data.txt內容   

### 參考資料

![Get user metadata](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/getmetadata.png?raw=true)    

```
https://www.itread01.com/articles/1490774251.html
```   
```
https://zhangchenchen.github.io/2017/01/13/openstack-init-instance-password/
```

### 未來我們的機制??



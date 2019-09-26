---
layout: post
title:  "Flask CORS"
date:   2019-09-27
excerpt: "介紹如何在Flask中處理CORS"
tag:
- Flask 
- CORS
comments: false
---  

### 跨來源資源共用（CORS:Cross-Origin Resource Sharing)

>现在前后端分离日益成为web开发主流方式的大趋势下，后台逐渐趋向指提供API服务，为各客户端提供数据及相关操作，而网站的开发全部交给前端搞定，网站和API服务很少部署在同一台服务器上并使用相同的端口，js的跨域请求时普遍存在的，开发RESTful API时，通常都要考虑到CORS功能的实现，以便js能正常使用API - 《RESTful API 编写指南》

 ![CORS](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/CORS-diagram.png?raw=true)   

 在後端伺服器收到request並且回傳resonse到client的時候，**瀏覽器**會去看response裡面的header–「Access-Control-Allow-Origin」是不是包含剛剛發出request寫的Origin 域名，如果有，資料才會允許被回傳。

 ### 使用案例

 透過Axios(HTTP函式庫)較用**GET /dashboard**:   

 ![axios call](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/vue_call.png?raw=true)   

 當Backend API尚未加上CORS處理時，取回的資料會被瀏覽器擋住:  

 ![Block](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/Block.jpg?raw=true) 

 ### 處理方式   

 安裝**Flask-CORS**套件，並在API方法上附加Attribute。   

 套件說明及安裝方式:   
 ```
 https://flask-cors.readthedocs.io/en/latest/
 ```   
 ![Backend CORS](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/backendapi_cors.PNG?raw=true)   

 再次透過axios叫用該方法:   

  ![Success](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080927/Success.jpg?raw=true)  





---
layout: post
title:  "HTTP通訊機制"
date:   2019-03-28
excerpt: "介紹HTTP常用的Verbs"
tag:
- HTTP 
comments: false
---

Web應用程式透過用戶端瀏覽器存取Web伺服器執行相關處理，此時用戶端與Web伺服器間的傳輸就採用HTTP通訊協定。  

HTTP是用於瀏覽器和Web伺服器之間發送和接收如HTML等內容的傳輸協定，又稱為**超文本傳輸協定**。是英國電腦科學家**Tim Berners-Lee**在1991年發明Web機制時開發出的協定，從HTTP/0.9開始到目前最新版本為HTTP/2，在2015年2月發表為規格。   

用戶端瀏覽器透過HTTP請求Web伺服器執行處理時會利用下表八個方法，當中**GET方法**與**POST方法**特別常用。

|方法|說明|
|---|---|
|GET|請求取得資源。瀏覽Web網站時可取得Web頁面/圖片   |
|POST|傳送表單新增資料到伺服器|
|PUT|請求資源更新|
|DELETE|請求資源刪除   |
|HEAD|請求HTTP標頭資訊   |
|CONNECT|透過代理伺服器進行SSL通訊   |
|OPTIONS|查詢伺服器支援的方法與選項   |
|TRACE|追蹤HTTP的動作   |


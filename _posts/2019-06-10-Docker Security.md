---
layout: post
title:  "Dock Security"
date:   2019-06-10
excerpt: "了解Docker環境的資訊安全脆弱點，並介紹一個自動掃描工具"
tag:
- Docker Security
- Docker 
- Docker Bench  
- Center for Internet Security 
comments: false
---

Docker技術火紅，讓應用程式的佈署/安裝門檻大幅降低，看到喜歡的應用程式想試，直接Pull下來再Docker Run就可拆箱即用。開發者也可以用其他人準備好的映像檔（Docker Image）作為基底製作客製化映像檔。但在這些操作中存在著資訊安全風險，有可能損害系統完整性或資料遭竊取。

根據Gartner，使用Docker技術有下面幾項主要挑戰︰
1. 多個容器共享一個OS
成功攻擊主機OS脆弱點將導致上面運行的所有容器無法正常運作。
</br>
2. 傳統Host-based的資安代理(Agent)能力有限
傳統代理不了解容器，亦不了解容器執行環境而無法對不同容器套用不同資安政策(Policy)。
</br>
3. 容器間通訊可視性受限
外部的網路防火牆、入侵偵測系統、入侵防禦系統無法檢查容器與容器間的通訊。
</br>
4. OS本身元件的脆弱點
作業系統本身存在的漏洞。
</br>

下面要架構性的方法思考使用Docker時環境會有哪些脆弱點及相對應的工具，或許可幫你安全的設置相關環境，以便放心的使用Docker。

[![Docker Securting Architecture](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080610/docker-architecture.png?raw=true)]


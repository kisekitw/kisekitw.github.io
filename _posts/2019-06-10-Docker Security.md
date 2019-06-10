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
2. 傳統Host-based的資安代理(Agent)能力有限
傳統代理不了解容器，亦不了解容器執行環境而無法對不同容器套用不同資安政策(Policy)。
3. 容器間通訊可視性受限
外部的網路防火牆、入侵偵測系統、入侵防禦系統無法檢查容器與容器間的通訊。
4. OS本身元件的脆弱點
作業系統本身存在的漏洞。

下面要架構性的方法思考使用Docker時環境會有哪些脆弱點及相對應的工具，或許可幫你安全的設置相關環境，以便放心的使用Docker︰

![Docker Securting Architecture](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080610/docker-architecture.png?raw=true)

* Securing Host 
1. Auto-patching hosts 
使用如CoreOS或Amazon Linux AMI這種會自行執行更新的作業系統，持續修補漏洞。
2. JEOS(Just Enough OS) 
移除作業系統中不必要的模組，保留最小且可運行的作業系統核心。
3. SELinux/AppArmor 
一個容器就可視為一個程序(Process)，利用SELinux/AppArmor為每個程序設置組態以限制程式功能。

* Securing Docker Daemon 
1. Certificates & Keys 
確保Docker Daemon只能給經認證的連線存取，可以產生Certificates&Keys作為連線認證使用。

* Securing Image 
1. Package Vulnerability Scanning 
利用Aqua Security的**MicroScanner**可檢驗容器映像檔的脆弱點，若你的映像檔中有任何已知的高風險項目，它會讓映像檔建制失敗。
2. Docker Content Trust 
透過Docker Content Trust可對建立好的映像檔作數位簽章認證。

* Securing Container 
1. Docker Security Scanning
Docker推出的容器漏洞掃描工具，可持續監控整個容器發展生命週期，在容器映像檔佈署前進行二進位掃描。
2. More "Assembled" than "Developed"
現今軟體開發模式組合OSS(Open Source Software)多於完全自行開發，透過**Black Duck Software**等供應商軟體可掃描使用的OSS是否存在有漏洞。
3. Systematic Workload Reprovisioning
使用腳本或模版定期性(系統性)的重新產生看起來健康的容器，藉此套用新的修補程式。

* Securing Registry 
1. Digital Signature
映像檔簽出/簽入時驗證數位簽章，確保佈署內容的完整性。
2. Docker Notary
Docker Notary確保所使用的映像檔與當初內容提供者是相同的。 

* Securing Runtime Environment 
1. Application Control Whitelisting
以白名單控制可在主機或容器內可運行的應用程式。
2. Security Vendors Supports 
請資訊安全廠商將其產品開放APIs，讓開發者可整合至CI/CD工具鍊。
3. VMs/Physical HW for Strong Isolation
透過將各容器跑在各自VM上或實體機器上，打造強隔離。 

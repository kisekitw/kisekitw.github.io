---
layout: post
title:  "Dock Security - PART II"
date:   2019-06-10
excerpt: "介紹一個Docker組態安全掃描工具"
tag:
- Docker Security
- Docker 
- Docker Bench  
- Center for Internet Security 
comments: false
---

前面以架構介紹了Docker環境中可能存在的脆弱點以及一些資安第三方工具，這邊要介紹一個非常方便的Docker組態的安全性掃描工具。首先就要先介紹一個非營利組織網路安全中心Center for Internet(CIS)，該單位的安全專業人員與主題專家針對每個平台的資訊安全最佳實務配置出版指南(Guide)，當中也有針對Docker Community出版指南，包含了下面各面向︰ 

1. Host Configuration 
2. Docker Daemon Configuration 
3. Docker Daemon Configuration Files
4. Container Images/Runtime 
5. Docker Security Operations

應該可以發現與先前說的架構的各面向非常相近。

指南的內容非常詳盡，例如下面針對Host Configuration的查核: 

![CIS Sample](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080610/CIS_Sample_1.png?raw=true)

描述該查核點的描述，如何執行，甚至會有何影響，若有興趣可到官方網站下載完整的指南︰

```
Center for Internet
https://www.cisecurity.org/
```

了解這些項目後，要如何知道自己的環境有哪些組態不符合最佳實務呢？最好還要能自動的檢查CIS這些面向！ 
答案是**Docker Bench Security**！ 

```
Docker Bench Security 
https://github.com/docker/docker-bench-security 
```

只要執行下面語法，就會自行檢測環境組態︰

```
sudo docker run -it --net host --pid host --cap-add audit_control -v /var/lib:/var/lib -v /var/run/docker.sock:/var/run/docker.sock -v /usr/lib/systemd:/usr/lib/systemd -v /etc:/etc --label docker_bench_security docker/docker-bench-security
```

執行的結果如下︰
![Docker Bench Result](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080610/DockerBenchResult.png?raw=true)

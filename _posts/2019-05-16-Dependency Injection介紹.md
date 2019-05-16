---
layout: post
title:  "Dependency Injection介紹"
date:   2019-05-16
excerpt: "Dependency Injection介紹"
tag:
- Design Pattern 
- DI 
- Dependency Injection
comments: false
---

[![Dependency Injection](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080516/DI_first.png?raw=true)](https://www.slideshare.net/kisekitw/dependency-injection-intro)

在架構與設計解決方案時，應謹記程式的可維護性，一般的設計原則如下：
1. 關注點分離 
2. 封裝 
3. 相依性反轉 

其中實做相依性反轉（Inversion of Control）的其中一種軟體設計模式就是Dependency Injection，這種模式目前被大量使用在各種框架（Framework）中，例如Dotnet Core、Angular等。 

投影片範例實做一個PrintService，當中用new物件的方式實做ConsoleOutput，這表示PrintService相依於ConsoleOutput。這時若需求更改為輸出至Text檔案，就必須修改PrintService並且重新編譯、測試，維護本很高，所代表的意思就是程式的耦合性（Coupling）過高。

為了降低耦合性，可以引入介面（Interface）的概念，針對介面寫程式。範例中建立一個介面IWrite，並讓ConsoleOutput、TextOutput實做（P.6），PrintService的部份則在建構子注入IWrite介面，換句話說，PrintService只與IWrite相依，而且該相依性是由上層模組給予。 

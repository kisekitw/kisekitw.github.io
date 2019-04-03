---
layout: post
title:  "JavaScript區塊鏈開發(二)"
date:   2019-04-03
excerpt: "建立專案並開始打造區塊鏈"
tag:
- JavaScript 
- Blockshain 
comments: false
---

[link]: https://kisekitw.github.io//JavaScript%E5%8D%80%E5%A1%8A%E9%8F%88%E9%96%8B%E7%99%BC(%E4%B8%80)/ "上一篇"

[上一篇][link]說明了何謂區塊鏈與其運作方式，接著要建立專案開始打造區塊鏈及其功能，包含:

1. 利用建構子(constructor)建立區塊鏈資料結構
2. 利用prototype實做區塊鏈各種方法，例如建立新區塊、雜湊資料、工作量證明(Proof of Work)等

### 專案環境設定

首先建立一個blackchain資料夾並進到該資料夾中，然後再建立一個dev資料夾, dev資料夾中新增blackchain.js、test.js檔案:

```
mkdir blackchain
cd blackchain
mkdir dev
cd dev
touch blackchain.js test.js
```

進到blackchain資料夾後，輸入下面指令進行專案初始化，所有選項都用初始值即可:

```
npm init
```

然後再建立一個dev資料夾, dev資料夾中新增blackchain.js、test.js檔案:

```
mkdir dev
cd dev
touch blackchain.js test.js
```

最後專案資料夾結構應該如下圖:

![alt text](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080403/projectstructure.png?raw=true "project file structure")














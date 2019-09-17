---
layout: post
title:  "Heroku Deploy"
date:   2019-09-17
excerpt: "介紹如何將程式佈署到Heroku平台上"
tag:
- Heroku 
- Deploy
comments: false
---  

1.  官網下載Heroku Cli，安裝heroku-x64.exe(For Windows)
    ```
    https://devcenter.heroku.com/articles/heroku-cli#download-and-install
    ```
2.  下載後執行安裝(也會安裝Git)，預設資料夾為:
    *  C:\Program Files\Heroku
    *  C:\Program Files\Git

3.  開啟Git Bash，輸入指令:
    ```
    $ heroku --version
    ```
4.  登入Heroku
    ```
    $ heroku login
    ```
    輸入帳號密碼

5.  檢查是否存在SSH Key Pair
    ```
    $ ls -a -l ~/.ssh
    ```
6.  產生SSH Key Pair
    ```
    $ ssh-keygen -t rsa -b 4096 -C "XXX@gmail.com"
    ```
7.  問項
    1.  預設
    2.  空白
    3.  空白

8.  再次檢查SSH Key Pair
    ```
    $ ls -a -l ~/.ssh
      total 29
      drwxr-xr-x 1 kisek 197609    0 七月    8  2018 ./
      drwxr-xr-x 1 kisek 197609    0 九月   16 16:33 ../
      -rw-r--r-- 1 kisek 197609 1675 七月    8  2018 id_rsa   <-- private key, never share to anyone
      -rw-r--r-- 1 kisek 197609  404 七月    8  2018 id_rsa.pub
    ```
9.  啟動SSH Agent
    ```
    $ eval "$(ssh-agent -s)"   
      Agent pid 17296
    ``` 
    有出現Agent pid資訊表示正常運行

10. 註冊SSH檔案
    For Windows、Linux:
    ```
    $ ssh-add ~/.ssh/id_rsa
      Identity added: /c/Users/kisek/.ssh/id_rsa (/c/Users/kisek/.ssh/id_rsa)
    ```
    For Mac:
    ```
    $ ssh-add -K ~/.ssh/id_rsa
    ```   
11. 設置SSH Public Key給Heroku
    ```
    $ heroku keys:add
       ?   Warning: heroku update available from 7.12.2 to 7.29.0
       Found an SSH public key at C:\Users\kisek\.ssh\id_rsa.pub
       Would you like to upload it to Heroku? [Y/n]: Y
       Uploading C:\Users\kisek\.ssh\id_rsa.pub SSH key... done
    ```   
12. 為專案建立Heroku專案名稱
    ```
    $ heroku create $uniqueName
    ```  
13. 修改package.json的script區塊(Option)
    ```json
    "start": "node src/app.js"
    ```
14. 修改web server監聽的Port(Option)

15. 檢查遠端儲存庫
    ```
    $ git remote
      heroku
      origin
    ```  
16. 上傳程式到Heroku
    ```
    $ git push heroku master
    ```

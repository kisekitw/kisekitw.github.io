---
layout: post
title:  "Openstack integration by gophercloud sdk (一)"
date:   2019-09-19
excerpt: "介紹如何透過gophercloud sdk呼叫openstack api"
tag:
- Golang 
- Gophercloud
- Openstack
comments: false
---  

### 身分驗證
在開始與Openstack服務互動前必須要先經過身分驗證，因此第一個步驟就是建構**gophercloud.AuthOptions**物件，裡面必須放入相關的驗證資訊，有兩種方式:   
1. 透過AuthOptionsFromEnv()方法
    可透過Openstack Dashboard下載shell scrip:   
     ![Openstack auth file download](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080919/openstack_auth_download.png?raw=true)   

     該檔案內容如下:   
     ``` shell
        #!/usr/bin/env bash
        # To use an OpenStack cloud you need to authenticate against the Identity
        # service named keystone, which returns a **Token** and **Service Catalog**.
        # The catalog contains the endpoints for all services the user/tenant has
        # access to - such as Compute, Image Service, Identity, Object Storage, Block
        # Storage, and Networking (code-named nova, glance, keystone, swift,
        # cinder, and neutron).
        #
        # *NOTE*: Using the 3 *Identity API* does not necessarily mean any other
        # OpenStack API is version 3. For example, your cloud provider may implement
        # Image API v1.1, Block Storage API v2, and Compute API v2.0. OS_AUTH_URL is
        # only for the Identity API served through keystone.
        export OS_AUTH_URL=http://rg1-vip.testbed-pike.local:5000/v3/
        # With the addition of Keystone we have standardized on the term **project**
        # as the entity that owns the resources.
        export OS_PROJECT_ID=8c7cb26f88774f15bc110d4e12c725ad
        export OS_PROJECT_NAME="rd-alston@test.com"
        export OS_USER_DOMAIN_NAME="Default"
        if [ -z "$OS_USER_DOMAIN_NAME" ]; then unset OS_USER_DOMAIN_NAME; fi
        export OS_PROJECT_DOMAIN_ID="default"
        if [ -z "$OS_PROJECT_DOMAIN_ID" ]; then unset OS_PROJECT_DOMAIN_ID; fi
        # unset v2.0 items in case set
        unset OS_TENANT_ID
        unset OS_TENANT_NAME
        # In addition to the owning entity (tenant), OpenStack stores the entity
        # performing the action as the **user**.
        export OS_USERNAME="rd-alston@test.com"
        # With Keystone you pass the keystone password.
        echo "Please enter your OpenStack Password for project $OS_PROJECT_NAME as user $OS_USERNAME: "
        OS_PASSWORD_INPUT="8323f15343239abb72885940220a4f3e"
        export OS_PASSWORD=$OS_PASSWORD_INPUT
        # If your configuration has multiple regions, we set that information here.
        # OS_REGION_NAME is optional and only valid in certain environments.
        export OS_REGION_NAME="RegionOne"
        # Don't leave a blank variable, unset it if it was empty
        if [ -z "$OS_REGION_NAME" ]; then unset OS_REGION_NAME; fi
        export OS_INTERFACE=public
        export OS_IDENTITY_API_VERSION=3
     ```   
     接著執行下面指令就會將相關資訊寫到環境變數中:   
     ```
     $ source auth.sh
     ```   
     如此在使用sdk時就可用下面語法進行驗證並生成**gophercloud.AuthOptions**物件:   
     ```golang
     import "github.com/rackspace/gophercloud/openstack"
     opts, err := openstack.AuthOptionsFromEnv()
     ```   
      
      PS: 測試沒成功，一直取到空值   

2. 自行建構   
    第二個方法就是自己建立**gophercloud.AuthOptions**物件，基本上就是登入Dashboard時所需要的資訊:   
    ```golang
    opts := gophercloud.AuthOptions{
		IdentityEndpoint: "http://rg1-vip.testbed-pike.local:5000/v3/",
		// Username:         "rd-alston@test.com", <-- 會驗證不過
		UserID:   "9f85b1bc11954d5d85e94fbbd600b85a", // <-- 驗證通過
		Password: "8323f15343239abb72885940220a4f3e",
		TenantID: "8c7cb26f88774f15bc110d4e12c725ad",
	}
    ```   

取得opts物件後，就要傳入**AuthenticatedClient**方法並取得**ProviderClient**結構:   
```golang
import "github.com/rackspace/gophercloud/openstack"

provider, err := openstack.AuthenticatedClient(opts)
```   
ProviderClient物件中有存取Openstack API相關的驗證、存取資訊，例如**token ID**、**Base URL**。

接著就可以利用ProviderClient派生下列各種資源的Client進行操作:   
* Compute   
* Object Storage   
* Networking   
* Block Storage   
* Identity   

### 



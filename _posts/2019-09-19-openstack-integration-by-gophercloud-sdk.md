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
        
        export OS_AUTH_URL=http://rg1-vip.testbed-pike.local:5000/v3/
        
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

接著就可以利用ProviderClient派生下列**各種資源的Client**進行操作:   
* Compute   
* Object Storage   
* Networking   
* Block Storage   
* Identity   

### 使用案例 - 創建VM執行個體   
```golang
import (
    ...
	"github.com/rackspace/gophercloud/openstack/compute/v2/servers"
)
func main() {
    // TODO: Authentication
    ...

    // TODO: Create Compute Client
    computeClient, err :=
		openstack.NewComputeV2(provider, gophercloud.EndpointOpts{
			Region: "RegionOne",
        })
        
    // TODO: Get Network List
    networkClient, err := openstack.NewNetworkV2(provider, gophercloud.EndpointOpts{
		Name:   "neutron",
		Region: "RegionOne",
    })
    
    ... 

	for _, network := range allNetworks {
		fmt.Printf("%+v", network)
    }
    
    // TODO: Create Server(VM)
    createOpts := servers.CreateOpts{
		Name:             "Build-From-Go", // Require
		FlavorName:       "m1.medium", // Option?
		ImageName:        "CloudPlatform-Ubuntu16.04", // Option?
		AvailabilityZone: "Availability-Zone-2", // Option?
		Networks: []servers.Network{
			servers.Network{UUID: "71821ba8-dbe1-474f-8957-503ad7f25128"}, // Option?
		},
	}
	server, err := servers.Create(computeClient, createOpts).Extract()

}
```

結果如下:   

![Openstack auth file download](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080919/openstack_vm_created.png?raw=true)   


### 補充
1. Server Create Options Struct   
    ```golang
    type CreateOpts struct {
        // Name [required] is the name to assign to the newly launched server.
        Name string

        // ImageRef [optional; required if ImageName is not provided] is the ID or full
        // URL to the image that contains the server's OS and initial state.
        // Also optional if using the boot-from-volume extension.
        ImageRef string

        // ImageName [optional; required if ImageRef is not provided] is the name of the
        // image that contains the server's OS and initial state.
        // Also optional if using the boot-from-volume extension.
        ImageName string

        // FlavorRef [optional; required if FlavorName is not provided] is the ID or
        // full URL to the flavor that describes the server's specs.
        FlavorRef string

        // FlavorName [optional; required if FlavorRef is not provided] is the name of
        // the flavor that describes the server's specs.
        FlavorName string

        // SecurityGroups [optional] lists the names of the security groups to which this server should belong.
        SecurityGroups []string

        // UserData [optional] contains configuration information or scripts to use upon launch.
        // Create will base64-encode it for you.
        UserData []byte

        // AvailabilityZone [optional] in which to launch the server.
        AvailabilityZone string

        // Networks [optional] dictates how this server will be attached to available networks.
        // By default, the server will be attached to all isolated networks for the tenant.
        Networks []Network

        // Metadata [optional] contains key-value pairs (up to 255 bytes each) to attach to the server.
        Metadata map[string]string

        // Personality [optional] includes files to inject into the server at launch.
        // Create will base64-encode file contents for you.
        Personality Personality

        // ConfigDrive [optional] enables metadata injection through a configuration drive.
        ConfigDrive bool

        // AdminPass [optional] sets the root user password. If not set, a randomly-generated
        // password will be created and returned in the response.
        AdminPass string

        // AccessIPv4 [optional] specifies an IPv4 address for the instance.
        AccessIPv4 string

        // AccessIPv6 [optional] specifies an IPv6 address for the instance.
        AccessIPv6 string
    }
    ```



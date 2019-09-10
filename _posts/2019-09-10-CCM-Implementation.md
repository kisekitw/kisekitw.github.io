---
layout: post
title:  "Kubernetes Cloud Controller Manager(CCM)功能實作"
date:   2019-09-10
excerpt: "介紹Kubernetes Cloud Controller Manager與OpenStack的實作方式"
tag:
- Kubernetes 
- Cloud Controller Manager
- OpenStack
comments: false
---  

### 前言    

先前談到KCM(Kubernetes Controller Manager)中與雲端相關的元件及功能描述如下:    

1. 節點控制器(Node Controller)   
    * **Initialize a node** with cloud specific zone/region labels
    * **Initialize a node** with cloud specific instance details, for example, type and size
    * Obtain the node’s network addresses and hostname
    * In case a node becomes unresponsive, check the cloud to see if the node has been deleted from the cloud. If the node has been deleted from the cloud, delete the Kubernetes Node object
2. 路由控制器(Route Controller)   
    * Responsible for configuring routes in the cloud appropriately so that containers on different nodes in the Kubernetes cluster can communicate with each other  
    * **Only** applicable for Google Compute Engine clusters
3. 服務控制器(Service Controller)   
    * Responsible for listening to service create, update, and delete events  
    * Configures **cloud load balancers** to reflect the state of the services in Kubernetes
4. 卷控制器(Volume Controller)   

Cloud Controller Manager(CCM)包含前三項，最後的Volume Controller因其涉及的複雜性與現有供應商獨特卷的實作邏輯而獨立於CCM外。  

為了要實作相關功能，供應商實作前必須先載入相依套件，即K8S提供的**k8s.io/cloud-provide**，github網址如下:   

```
https://github.com/kubernetes/kubernetes/tree/master/staging/src/k8s.io/cloud-provider   
```
關於介面的制定都由**Sig-Cloudprovider**維護。

### 以節點控制器實作為例子   
OpenStack提供的CCM實作都放在**pkg/cloudprovider/providers/openstack**下:   

![Openstack CCM Provider Folder](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080910/OpenstackCCMFolder.png?raw=true)    

Kubernetes CCM要求的介面實作方法大部分都在**openstack.go**中，例如節點相關的實作則放在**openstack_instances.go**內。

不管在哪個檔案實作，都要先在Import區塊載入包含介面的套件:   

```golang
...   
cloudprovider "k8s.io/cloud-provider"   
...   
```   

在**openstack_instances.go**中除了實作**Interface**介面外，還實作**Instances**介面，有下面方法要實作:

* NodeAddresses(ctx context.Context, name types.NodeName)      
    回傳特定實體位址   
* NodeAddressesByProviderID(ctx context.Context, providerID string)      
    依節點的Provider ID回傳特定實體位址   
* InstanceID(ctx context.Context, nodeName types.NodeName)    
    依特定節點名稱回傳節點的Cloud Provider ID   
* InstanceType(ctx context.Context, name types.NodeName)   
    回傳特定實體型態   
* InstanceTypeByProviderID(ctx context.Context, providerID string)   
    依Provider ID回傳特定實體型態
* AddSSHKeyToAllInstances()   
    為所有實體新增SSH Key
* CurrentNodeName(ctx context.Context, hostname string)      
    回傳目前正在運行的節點名稱
* InstanceExistsByProviderID(ctx context.Context, providerID string)   
    依所提供的Provider ID若仍存在實體則回傳true，反之則回傳false並直接由Cloud Controller Manager刪除。對於存在但狀態為停止/休眠的實例，此方法仍應回傳true   
* InstanceShutdownByProviderID(ctx context.Context, providerID string)     
    回傳true表示該實體被cloudprovider關機   

    ```   
    https://github.com/kubernetes/cloud-provider-openstack/blob/master/pkg/cloudprovider/providers/openstack/openstack_client.go
    ```

從上面各項實作可知到兩件事情:   
1. 這些方法都為了提供K8S獲取相關OpenStack實體資訊，並無從K8S請求OpenStack創建實體的方法。  
2. 與OpenStack元件API的互動都透過**Gophercloud SDK**   

### Gophercloud SDK   

Gophercloud SDK是一套用Golang開發的工具，可讓應用程式與OpenStack Cloud溝通，支援OpenStack各種服務，例如:  
* Compute   
* Object Storage   
* Indentity   
* Networking   
* Block Storage   

```
http://gophercloud.io/
```   

### Gophercloud SDK實作   

1. 身分驗證

    ``` golang
    // TODO: 準備驗證相關資訊(使用rd-alston@test.com)
	opts := gophercloud.AuthOptions{
		IdentityEndpoint: "http://rg1-vip.testbed-pike.local:5000/v3/",
		// Username:         "rd-alston@test.com", 使用Username驗證會失敗
		UserID:   "9f85b1bc11954d5d85e94fbbd600b85a",
		Password: "8323f15343239abb72885940220a4f3e",
		TenantID: "8c7cb26f88774f15bc110d4e12c725ad",
    }
    
    provider, err := openstack.AuthenticatedClient(opts)
    ```
2. 創建Compute服務Client
    為了使用Compute API，可將取得的provider注入該服務中取得client物件。   

    ```golang   
    client, err :=
		openstack.NewComputeV2(provider, gophercloud.EndpointOpts{
			Region: "RegionOne",
		})
    ```   
    該client就可用於任何想要的Compute API操作。

3. 取得目前運行的實體資訊   

    ```golang
    opts2 := servers.ListOpts{}
	pager := servers.List(client, opts2)

	pager.EachPage(func(page pagination.Page) (bool, error) {
		serverList, err := servers.ExtractServers(page)

		if err != nil {
			return false, err
		}

		for _, s := range serverList {

			fmt.Println(s.ID, s.Name, s.Status)
			// servers.Delete(client, s.ID);
		}
		return true, nil
	})
    ```   

    ```cmd
    $ go run main.go
    52456aa3-c675-478b-981e-3902aa7f617c testvm ACTIVE
    ```

### Customer Controller的可能性   



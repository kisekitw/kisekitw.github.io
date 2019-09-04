---
layout: post
title:  "Kubernetes Cloud Controller Manager(CCM)整合方法"
date:   2019-09-04
excerpt: "介紹Kubernetes Cloud Controller Manager與OpenStack整合方式"
tag:
- Kubernetes 
- Cloud Controller Manager
- OpenStack
comments: false
---  
### In Tree ?
K8S v1.4版的原始碼內置很多的Cloud Provider實作，當下載K8S後預設可透過配置直接使用這些Cloud Provider:

1. OpenStack
2. AWS
3. AZURE
4. GCE
5. ...

 ![K8S v1.4 Cloud Provider](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080904/k8s14CloudProvider.png?raw=true)

```
https://github.com/kubernetes/kubernetes/tree/release-1.14/pkg/cloudprovider/providers
```

### 內置Cloud Provider的問題

1. K8S的核心能力應專注為一個指揮調度的工具，這些不同的雲端供應商應由各供應商自行維護
2. 對於特定的佈署情況，不需要下載用不到的Provider
3. 這些Provider的重大更新耦合K8S更新周期
4. 某些Provier已暫停更新

### 如何提供固定的Cloud Provider?
K8S創建一個Cloud Controller Manager Interface給外部(Out of tree)雲端供應商實作

```golang
// Interface is an abstract, pluggable interface for cloud providers.
type Interface interface {
	// Initialize provides the cloud with a kubernetes client builder and may spawn goroutines
	// to perform housekeeping or run custom controllers specific to the cloud provider.
	// Any tasks started here should be cleaned up when the stop channel closes.
	Initialize(clientBuilder ControllerClientBuilder, stop <-chan struct{})
	// LoadBalancer returns a balancer interface. Also returns true if the interface is supported, false otherwise.
	LoadBalancer() (LoadBalancer, bool)
	// Instances returns an instances interface. Also returns true if the interface is supported, false otherwise.
	Instances() (Instances, bool)
	// Zones returns a zones interface. Also returns true if the interface is supported, false otherwise.
	Zones() (Zones, bool)
	// Clusters returns a clusters interface.  Also returns true if the interface is supported, false otherwise.
	Clusters() (Clusters, bool)
	// Routes returns a routes interface along with whether the interface is supported.
	Routes() (Routes, bool)
	// ProviderName returns the cloud provider ID.
	ProviderName() string
	// HasClusterID returns true if a ClusterID is required and set
	HasClusterID() bool
}
```   
只有**ProviderName()**為必要實作，其他皆為Option。

###  Cloud Controller Manager 
未使用CCM的K8S叢集架構:   
![K8S CCM Old Infra](https://d33wubrfki0l68.cloudfront.net/e298a92e2454520dddefc3b4df28ad68f9b91c6f/70d52/images/docs/pre-ccm-arch.png?raw=true)    

使用CCM的K8S叢集架構:   
![K8S CCM New Infra](https://d33wubrfki0l68.cloudfront.net/518e18713c865fe67a5f23fc64260806d72b38f5/61d75/images/docs/post-ccm-arch.png?raw=true)      

CCM從Kubernetes controller manager(KCM)中分離出與雲端供應商相關的功能元件:

* Node Controller
* Route Controller
* Service Controller
* Volume Controller

在K8S v1.9中實際運行的為上面前三個，Volume的部分由於抽象化各供應商Volume邏輯過於複雜，目前決定不將其加入CCM。

### 創建客制化CCM

1. Create a go package that satisfies **the cloud provider interface**   
2. Create a copy of the Cloud Controller Manager **main.go** and **import your package**, making sure there is an init block available      
    ```golang
    import (
    "fmt"
    "math/rand"
    "os"
    "time"
    "k8s.io/component-base/logs"
    "k8s.io/kubernetes/cmd/cloud-controller-manager/app"
    "<my_cloud_provider>"
    _ "k8s.io/kubernetes/pkg/util/prometheusclientgo" // load all the prometheus client-go plugins
    _ "k8s.io/kubernetes/pkg/version/prometheus" // for version metric registration
    )
    func main() {
    rand.Seed(time.Now().UnixNano())
    command := app.NewCloudControllerManagerCommand()
    // TODO: once we switch everything over to Cobra commands, we can go back to calling
    // utilflag.InitFlags() (by removing its pflag.Parse() call). For now, we have to set the
    // normalize func and add the go flag set by hand.
    // utilflag.InitFlags()
    logs.InitLogs()
    defer logs.FlushLogs()
    if err := command.Execute(); err != nil {
    fmt.Fprintf(os.Stderr, "error: %v\n", err)
    os.Exit(1)
    }
    }
    ```   

### UseCase : Cloud Provider OpenStack
1. 實作Cloud Provider Interface  
    ```golang   

    ...   

    // Clusters is a no-op   
    func (os *OpenStack) Clusters() (cloudprovider.Clusters, bool) {
        return nil, false
    }   

    // ProviderName returns the cloud provider ID.
    func (os *OpenStack) ProviderName() string {
        return ProviderName
    }   

    // HasClusterID returns true if the cluster has a clusterID
    func (os *OpenStack) HasClusterID() bool {
        return true
    }

    // LoadBalancer initializes a LbaasV2 object
    func (os *OpenStack) LoadBalancer() (cloudprovider.LoadBalancer, bool) {

        ...
        return &LbaasV2{LoadBalancer{network, compute, lb, os.lbOpts}}, true
    }   

    // Zones indicates that we support zones
    func (os *OpenStack) Zones() (cloudprovider.Zones, bool) {

        klog.V(1).Info("Claiming to support Zones")

        return os, true

    }
    ```  

    ```
    https://github.com/kubernetes/cloud-provider-openstack/blob/master/pkg/cloudprovider/providers/openstack/openstack.go
    ```   

2. 在main.go中引用該套件   
    ```golang
    import (
    ...   

    "k8s.io/cloud-provider-openstack/pkg/cloudprovider/providers/openstack"
    
    ...

    _ "k8s.io/kubernetes/pkg/version/prometheus" // for version metric registration
    
    ...
    )
    ```   

    ```
    https://github.com/kubernetes/cloud-provider-openstack/blob/master/cmd/openstack-cloud-controller-manager/main.go
    ```
3. 製作Docker Image
    ```Dockerfile   
    FROM alpine:3.7
    RUN apk add --no-cache ca-certificates
    ADD openstack-cloud-controller-manager /bin/
    CMD ["/bin/openstack-cloud-controller-manager"]   
    ```     
         
    ```   
    https://github.com/kubernetes/cloud-provider-openstack/blob/master/cluster/images/controller-manager/Dockerfile   
    ```   

4. 以DaemonSet掛載至Master Node   
    ```yaml
    apiVersion: apps/v1
    kind: DaemonSet
    metadata:
    name: openstack-cloud-controller-manager
    namespace: kube-system
    labels:
        k8s-app: openstack-cloud-controller-manager
    spec:
    selector:
        matchLabels:
        k8s-app: openstack-cloud-controller-manager
    updateStrategy:
        type: RollingUpdate
    template:
        metadata:
        labels:
            k8s-app: openstack-cloud-controller-manager
        spec:
        nodeSelector:
            node-role.kubernetes.io/master: ""
        securityContext:
            runAsUser: 1001
        tolerations:
        - key: node.cloudprovider.kubernetes.io/uninitialized
            value: "true"
            effect: NoSchedule
        - key: node-role.kubernetes.io/master
            effect: NoSchedule
        serviceAccountName: cloud-controller-manager
        containers:
            - name: openstack-cloud-controller-manager
            image: docker.io/k8scloudprovider/openstack-cloud-controller-manager:latest
        
        ...   
    ```   

    ```
    https://github.com/kubernetes/cloud-provider-openstack/blob/master/manifests/controller-manager/openstack-cloud-controller-manager-ds.yaml
    ```   

    ![Openstack CCM Image](https://github.com/kisekitw/kisekitw.github.io/blob/master/assets/img/1080904/OpenstackCCMImage.png?raw=true)


    ```
    https://hub.docker.com/r/k8scloudprovider/openstack-cloud-controller-manager/tags
    ```

### 參考資料

* Concepts Underlying the Cloud Controller Manager   
https://kubernetes.io/docs/concepts/architecture/cloud-controller/  

* Cloud Provider OpenStack   
https://github.com/kubernetes/cloud-provider-openstack/   

* Building a Controller Manager for Your Cloud Platform   
https://www.youtube.com/watch?v=kO7qJKPgxS0  



* OCI Cloud Controller Manager (CCM)   
https://github.com/oracle/oci-cloud-controller-manager  





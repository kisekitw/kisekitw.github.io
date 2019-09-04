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
只有***ProviderName()***為必要實作，其他皆為Option。

### 

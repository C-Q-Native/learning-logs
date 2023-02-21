---
title: "从日志来探究Dapr源码-Dapr runtime的启动过程"
date: 2023-01-13T23:02:40+08:00
categories:
- Dapr
tags:
- Dapr
- cloud-native
- kubernetes
---

#### 日志
首先，我们来整体观察一下在kubernetes集群环境下dapr服务的启动日志，以便对启动过程的时间线有一个整体的印象。
<!--more-->
```
time="2022-05-12T01:51:07.584216594Z" level=info msg="starting Dapr Runtime -- version 1.7.1 -- commit 5fd38aeaaa9cf58f3f00f6f9bfc6c58d8cd82b87" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.584279407Z" level=info msg="log level set to: debug" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.585832265Z" level=info msg="metrics server started on :9090/" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.metrics type=log ver=1.7.1
time="2022-05-12T01:51:07.585962686Z" level=info msg="loading default configuration" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.585982324Z" level=debug msg="Resiliency is not enabled." app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.586012006Z" level=info msg="kubernetes mode configured" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.586025078Z" level=info msg="app id: nodeapp" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.586393412Z" level=info msg="mTLS enabled. creating sidecar authenticator" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.588174923Z" level=info msg="trust anchors and cert chain extracted successfully" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.security type=log ver=1.7.1
time="2022-05-12T01:51:07.588201977Z" level=info msg="authenticator created" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.594025729Z" level=info msg="Initialized name resolution to kubernetes" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.597683105Z" level=debug msg="loading component. name: kubernetes, type: secretstores.kubernetes/v1" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.599609839Z" level=info msg="component loaded. name: kubernetes, type: secretstores.kubernetes/v1" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:07.599642778Z" level=info msg="loading components" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.571722227Z" level=debug msg="found component. name: statestore, type: state.redis/v1" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.575529535Z" level=info msg="waiting for all outstanding components to be processed" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.575745717Z" level=debug msg="loading component. name: statestore, type: state.redis/v1" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.599485999Z" level=info msg="component loaded. name: statestore, type: state.redis/v1" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.599538024Z" level=info msg="all outstanding components processed" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.599558821Z" level=info msg="gRPC proxy enabled" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.599651105Z" level=warning msg="Failed to listen on [::1]:50001 with error: listen tcp [::1]:50001: bind: cannot assign requested address" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.api type=log ver=1.7.1
time="2022-05-12T01:51:10.599707846Z" level=info msg="enabled gRPC tracing middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.api type=log ver=1.7.1
time="2022-05-12T01:51:10.599733291Z" level=info msg="enabled gRPC metrics middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.api type=log ver=1.7.1
time="2022-05-12T01:51:10.604608235Z" level=info msg="API gRPC server is running on port 50001" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.604983466Z" level=info msg="enabled metrics http middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.http type=log ver=1.7.1
time="2022-05-12T01:51:10.605010379Z" level=info msg="enabled tracing http middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.http type=log ver=1.7.1
time="2022-05-12T01:51:10.605067197Z" level=warning msg="Failed to listen on [::1]:3500 with error: listen tcp [::1]:3500: bind: cannot assign requested address" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.http type=log ver=1.7.1
time="2022-05-12T01:51:10.605171418Z" level=info msg="enabled metrics http middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.http type=log ver=1.7.1
time="2022-05-12T01:51:10.605187231Z" level=info msg="enabled tracing http middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.http type=log ver=1.7.1
time="2022-05-12T01:51:10.605217462Z" level=info msg="http server is running on port 3500" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.605226165Z" level=info msg="The request body size parameter is: 4" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.605254349Z" level=info msg="enabled gRPC tracing middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.internal type=log ver=1.7.1
time="2022-05-12T01:51:10.605270933Z" level=info msg="enabled gRPC metrics middleware" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.internal type=log ver=1.7.1
time="2022-05-12T01:51:10.605285517Z" level=info msg="sending workload csr request to sentry" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.internal type=log ver=1.7.1
time="2022-05-12T01:51:10.693861196Z" level=info msg="certificate signed successfully" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.internal type=log ver=1.7.1
time="2022-05-12T01:51:10.694073123Z" level=info msg="internal gRPC server is running on port 50002" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.694093279Z" level=info msg="application protocol: http. waiting on port 3000.  This will block until the app is listening on that port." app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.694812954Z" level=info msg="starting workload cert expiry watcher. current cert expires on: 2022-05-13 02:06:10 +0000 UTC" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime.grpc.internal type=log ver=1.7.1
time="2022-05-12T01:51:10.696089494Z" level=info msg="application discovered on port 3000" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.696129066Z" level=warning msg="[DEPRECATION NOTICE] Adding a default content type to incoming service invocation requests is deprecated and will be removed in the future. See https://docs.dapr.io/operations/support/support-preview-features/ for more details. You can opt into the new behavior today by setting the configuration option `ServiceInvocation.NoDefaultContentType` to true." app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.82914006Z" level=info msg="application configuration loaded" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.82927049Z" level=warning msg="failed to init actors: no actor state store defined" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
time="2022-05-12T01:51:10.829303428Z" level=info msg="dapr initialized. Status: Running. Init Elapsed 3243.280519ms" app_id=nodeapp instance=nodeapp-967854b66-vdcwt scope=dapr.runtime type=log ver=1.7.1
```
（如何在k8s集群初始化dapr，如何跑demo，请[参照文档](https://github.com/dapr/quickstarts/tree/master/tutorials/hello-kubernetes)）
以上，即为demo镜像完整的启动日志。接下来，我们来尝试根据启动日志，逐步分析dapr runtime在启动过程中都做了哪些工作。

#### Sidecar注入
在demo构建模板yaml文件里有这样几行代码：
```
annotations:
   dapr.io/enabled: "true"
   dapr.io/app-id: "nodeapp"
   dapr.io/app-port: "3000"
   dapr.io/enable-api-logging: "true"
```
服务在构建时，Dapr控制面板会根据以上配置项在宿主pod中自动注入sidecar。

#### 启动前准备
dapr启动入口代码在`dapr/pkg/runtime/cli.go`文件中实现：

**开端**
```
msg="starting Dapr Runtime"
msg="log level set to: debug"
```
这两行日志预示着dapr runtime正式开始启动，在FromFlags()方法中开启了runtime启动的漫长过程。

```
func FromFlags() (*DaprRuntime, error) {
...
	解析启动配置参数，为runtime启动初始化做准备
	log.Infof("starting Dapr Runtime -- version %s -- commit %s", version.Version(), version.Commit())
	log.Infof("log level set to: %s", loggerOptions.OutputLevel)

	#初始化dapr metrics
	if err := metricsExporter.Init(); err != nil {
		log.Fatal(err)
	}
	
	#加载resiliency配置
	if resiliencyEnabled {
		
	} else {
		log.Debug("Resiliency is not enabled.")
	}
...
}
```

**Metrics**
```
msg="metrics server started"
```
在文件`pkg/metrics/exporter.go`中实现了metrics server启动的全部过程：

```
#初始化opencensus exporter
func (m *promMetricsExporter) Init() error {	
	...
	// start metrics server
	return m.startMetricServer()
}

#启动metrics server
func (m *promMetricsExporter) startMetricServer() error {

  addr := fmt.Sprintf(":%d", m.options.MetricsPort())

  m.exporter.logger.Infof("metrics server started on %s%s", addr, defaultMetricsPath)
  
  go func() {
    mux := http.NewServeMux()
    mux.Handle(defaultMetricsPath, m.ocExporter)

    if err := http.ListenAndServe(addr, mux); err != nil {
      m.exporter.logger.Fatalf("failed to start metrics server: %v", err)
    }
  }()

  return nil
}
```

**Resiliency**
```
msg="Resiliency is not enabled."
```
Resiliency为1.7版本的新特性，低版本的代码中没有这部分处理逻辑。验证是否在spec中声明支持resiliency特性，并且根据mode加载resiliency配置。

```
func FromFlags() (*DaprRuntime, error) {
...
	if resiliencyEnabled {
		#加载配置
	} else {
		log.Debug("Resiliency is not enabled.")
	}
...
}
```
**runtime实例**
根据传入的runtime配置参数和全局的默认参数创建一个runtime实例。
```
func FromFlags() (*DaprRuntime, error) {
...
	return NewDaprRuntime(runtimeConfig, globalConfig, accessControlList, resiliencyProvider), nil
}

func NewDaprRuntime(runtimeConfig *Config, globalConfig *config.Configuration, accessControlList *config.AccessControlList, resiliencyProvider resiliency.Provider) *DaprRuntime {
	return &DaprRuntime{
...
	}
}
```
#### 启动
```
msg="kubernetes mode configured" 
msg="app id: nodeapp" 
```
经过前置的准备工作，runtime实例拿到了所有启动需要的配置参数，正式进入启动阶段。dapr 启动函数在`pkg/runtime/runtime.go`中实现，根据选择的运行模式开始启动。
```
func (a *DaprRuntime) Run(opts ...Option) error {
	log.Infof("%s mode configured", a.runtimeConfig.Mode)
	log.Infof("app id: %s", a.runtimeConfig.ID)
	err := a.initRuntime(&o)
	...
}

func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
	#如果显示声明了MetricSpec，则会初始化默认的服务监控、GRPC、HTTP和组件特性的监控指标
	if a.globalConfig.Spec.MetricSpec.Enabled {
		diag.InitMetrics(a.runtimeConfig.ID, a.namespace);
	}
	
	#建立mTLS安全认证
    err := a.establishSecurity(a.runtimeConfig.SentryServiceAddress)
    
	#为服务发现向注册表中注册和实例化名字解析组件
	#目前支持三种解析器：mdns，kubernetes，consul
	a.nameResolutionRegistry.Register(opts.nameResolutions...)
	err = a.initNameResolution()
}
```

**安全认证**

```
msg="mTLS enabled. creating sidecar authenticator"
msg="trust anchors and cert chain extracted successfully"
msg="authenticator created"
```
在方法establishSecurity()中完成了一系列的安全认证初始化操作

```
func (a *DaprRuntime) establishSecurity(sentryAddress string) error {
    log.Info("mTLS enabled. creating sidecar authenticator")
	#根据配置的证书创建Authenticator实例
	auth, err := security.GetSidecarAuthenticator(sentryAddress, a.runtimeConfig.CertChain)
	#注册runtime认证器
	a.authenticator = auth
	a.grpc.SetAuthenticator(auth)
	log.Info("authenticator created")
	diag.DefaultMonitoring.MTLSInitCompleted()
	return nil
}
```
**名称解析**

```
msg="Initialized name resolution to kubernetes"
```

```
func (a *DaprRuntime) initNameResolution() error {
	if resolverName == "" {
		switch a.runtimeConfig.Mode {
		case modes.KubernetesMode:
			resolverName = "kubernetes"
		case modes.StandaloneMode:
			resolverName = "mdns"
		default:
			return errors.Errorf("unable to determine name resolver for %s mode", string(a.runtimeConfig.Mode))
		}
	}
	#根据解析器类型和版本创建解析器实例
	resolver, err = a.nameResolutionRegistry.Create(resolverName, resolverVersion)
	#初始化解析器实例
	if err = resolver.Init(resolverMetadata); err != nil {
		log.Errorf("failed to initialize name resolution resolver %s: %s", resolverName, err)
		return err
	}
	log.Infof("Initialized name resolution to %s", resolverName)
	return nil
}
```

**组件**


```
msg="loading component. name: kubernetes, type: secretstores.kubernetes/v1"
msg="component loaded. name: kubernetes, type: secretstores.kubernetes/v1"
msg="loading components"
msg="found component. name: statestore, type: state.redis/v1"
msg="waiting for all outstanding components to be processed"
msg="loading component. name: statestore, type: state.redis/v1"
msg="component loaded. name: statestore, type: state.redis/v1"
msg="all outstanding components processed" 
```


将启动入口声明的组件注册到runtime 实例中。
```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
	...
	#向注册表中注册各种组件
	a.pubSubRegistry.Register(opts.pubsubs...)
	a.secretStoresRegistry.Register(opts.secretStores...)
	a.stateStoreRegistry.Register(opts.states...)
	a.configurationStoreRegistry.Register(opts.configurations...)
	a.bindingsRegistry.RegisterInputBindings(opts.inputBindings...)
	a.bindingsRegistry.RegisterOutputBindings(opts.outputBindings...)
	a.httpMiddlewareRegistry.Register(opts.httpMiddleware...)
	
	#加载处理各种组件
	go a.processComponents()

	if _, ok := os.LookupEnv(hotReloadingEnvVar); ok {
		log.Debug("starting to watch component updates")
		err = a.beginComponentsUpdates()
		if err != nil {
			log.Warnf("failed to watch component updates: %s", err)
		}
	}

	a.appendBuiltinSecretStore()
	#加载组件
	err = a.loadComponents(opts)
	#将未就绪组件刷新进管道
	a.flushOutstandingComponents()
	...
}

func (a *DaprRuntime) processComponents() {
	for comp := range a.pendingComponents {
		if comp.Name == "" {
			continue
		}

		err := a.processComponentAndDependents(comp)
		if err != nil {
			e := fmt.Sprintf("process component %s error: %s", comp.Name, err.Error())
			if !comp.Spec.IgnoreErrors {
				log.Warnf("process component error daprd process will exited, gracefully to stop")
				a.Shutdown(a.runtimeConfig.GracefulShutdownDuration)
				log.Fatalf(e)
			}
			log.Errorf(e)
		}
	}
}

func (a *DaprRuntime) processComponentAndDependents(comp components_v1alpha1.Component) error {
	log.Debugf("loading component. name: %s, type: %s/%s", comp.ObjectMeta.Name, comp.Spec.Type, comp.Spec.Version)
	res := a.preprocessOneComponent(&comp)
	if res.unreadyDependency != "" {
		a.pendingComponentDependents[res.unreadyDependency] = append(a.pendingComponentDependents[res.unreadyDependency], comp)
		return nil
	}
	compCategory := a.extractComponentCategory(comp)
	ch := make(chan error, 1)

	timeout, err := time.ParseDuration(comp.Spec.InitTimeout)
	if err != nil {
		timeout = defaultComponentInitTimeout
	}

	go func() {
		ch <- a.doProcessOneComponent(compCategory, comp)
	}()

	select {
	case err := <-ch:
		if err != nil {
			return err
		}
	case <-time.After(timeout):
		return fmt.Errorf("init timeout for component %s exceeded after %s", comp.Name, timeout.String())
	}

	log.Infof("component loaded. name: %s, type: %s/%s", comp.ObjectMeta.Name, comp.Spec.Type, comp.Spec.Version)
	a.appendOrReplaceComponents(comp)
	diag.DefaultMonitoring.ComponentLoaded()

	dependency := componentDependency(compCategory, comp.Name)
	if deps, ok := a.pendingComponentDependents[dependency]; ok {
		delete(a.pendingComponentDependents, dependency)
		for _, dependent := range deps {
			if err := a.processComponentAndDependents(dependent); err != nil {
				return err
			}
		}
	}

	return nil
}

#加载组件
func (a *DaprRuntime) loadComponents(opts *runtimeOpts) error {
	var loader components.ComponentLoader

	switch a.runtimeConfig.Mode {
	case modes.KubernetesMode:
		loader = components.NewKubernetesComponents(a.runtimeConfig.Kubernetes, a.namespace, a.operatorClient, a.podName)
	case modes.StandaloneMode:
		loader = components.NewStandaloneComponents(a.runtimeConfig.Standalone)
	default:
		return errors.Errorf("components loader for mode %s not found", a.runtimeConfig.Mode)
	}

	log.Info("loading components")
	comps, err := loader.LoadComponents()
	if err != nil {
		return err
	}
	for _, comp := range comps {
		log.Debugf("found component. name: %s, type: %s/%s", comp.ObjectMeta.Name, comp.Spec.Type, comp.Spec.Version)
	}

	authorizedComps := a.getAuthorizedComponents(comps)

	a.componentsLock.Lock()
	a.components = make([]components_v1alpha1.Component, len(authorizedComps))
	copy(a.components, authorizedComps)
	a.componentsLock.Unlock()

	for _, comp := range authorizedComps {
		a.pendingComponents <- comp
	}

	return nil
}
```
**GRPC Server**
```
msg="gRPC proxy enabled"
msg="enabled gRPC tracing middleware" 
msg="enabled gRPC metrics middleware"
msg="API gRPC server is running on port 50001"
```
Dapr runtime提供了Dapr HTTP API 和 Dapr gRPC API两套API。Dapr gRPC API的初始化和启动：

```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	// Start proxy，为服务调用信息投递建立GRPC代理连接
	a.initProxy()
    // Create and start internal and external gRPC servers
	grpcAPI := a.getGRPCAPI()

	err = a.startGRPCAPIServer(grpcAPI, a.runtimeConfig.APIGRPCPort)
...
}

func (a *DaprRuntime) initProxy() {
	a.proxy = messaging.NewProxy(a.grpc.GetGRPCConnection, a.runtimeConfig.ID,
		fmt.Sprintf("%s:%d", channel.DefaultChannelAddress, a.runtimeConfig.ApplicationPort), a.runtimeConfig.InternalGRPCPort, a.accessControlList, a.runtimeConfig.AppSSL, a.resiliency)

	log.Info("gRPC proxy enabled")
}

func (a *DaprRuntime) getGRPCAPI() grpc.API {
	return grpc.NewAPI(a.runtimeConfig.ID, a.appChannel, a.resiliency, a.stateStores, a.secretStores, a.secretsConfiguration, a.configurationStores,
		a.getPublishAdapter(), a.directMessaging, a.actor,
		a.sendToOutputBinding, a.globalConfig.Spec.TracingSpec, a.accessControlList, string(a.runtimeConfig.ApplicationProtocol), a.getComponents, a.ShutdownWithWait)
}

func (a *DaprRuntime) startGRPCAPIServer(api grpc.API, port int) error {
	serverConf := a.getNewServerConfig(a.runtimeConfig.APIListenAddresses, port)
	
	#获取一个面向用户侧的gRPC API server 对象
	server := grpc.NewAPIServer(api, serverConf, a.globalConfig.Spec.TracingSpec, a.globalConfig.Spec.MetricSpec, a.globalConfig.Spec.APISpec, a.proxy)
	if err := server.StartNonBlocking(); err != nil {
		return err
	}
	a.apiClosers = append(a.apiClosers, server)

	return nil
}
```

**HTTP Server**

```
msg="enabled metrics http middleware"
msg="enabled tracing http middleware"
msg="http server is running on port 3500"
msg="The request body size parameter is: 4"
```
Dapr HTTP API初始化和启动过程：

```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	// Start HTTP Server
	err = a.startHTTPServer(a.runtimeConfig.HTTPPort, a.runtimeConfig.PublicPort, a.runtimeConfig.ProfilePort, a.runtimeConfig.AllowedOrigins, pipeline)
	
...
}

func (a *DaprRuntime) startHTTPServer(port int, publicPort *int, profilePort int, allowedOrigins string, pipeline http_middleware.Pipeline) error {
	//注册http接口以及handler
	a.daprHTTPAPI = http.NewAPI(a.runtimeConfig.ID,
		a.appChannel,
		a.directMessaging,
		a.getComponents,
		a.resiliency,
		a.stateStores,
		a.secretStores,
		a.secretsConfiguration,
		a.getPublishAdapter(),
		a.actor,
		a.sendToOutputBinding,
		a.globalConfig.Spec.TracingSpec,
		a.ShutdownWithWait,
	)
...
	server := http.NewServer(a.daprHTTPAPI,
		serverConf, a.globalConfig.Spec.TracingSpec, a.globalConfig.Spec.MetricSpec, pipeline, a.globalConfig.Spec.APISpec)
	if err := server.StartNonBlocking(); err != nil {
		return err
	}
	a.apiClosers = append(a.apiClosers, server)

	return nil
}
```
API对象实例方法在`github.com/dapr/dapr/pkg/http/api.go`文件NewAPI()方法实现
```
// NewAPI returns a new API.
func NewAPI(
	appID string,
	appChannel channel.AppChannel,
	directMessaging messaging.DirectMessaging,
	getComponentsFn func() []components_v1alpha1.Component,
	resiliency resiliency.Provider,
	stateStores map[string]state.Store,
	secretStores map[string]secretstores.SecretStore,
	secretsConfiguration map[string]config.SecretsScope,
	pubsubAdapter runtime_pubsub.Adapter,
	actor actors.Actors,
	sendToOutputBindingFn func(name string, req *bindings.InvokeRequest) (*bindings.InvokeResponse, error),
	tracingSpec config.TracingSpec,
	shutdown func(),
) API {
	transactionalStateStores := map[string]state.TransactionalStore{}
	for key, store := range stateStores {
		if state.FeatureTransactional.IsPresent(store.Features()) {
			transactionalStateStores[key] = store.(state.TransactionalStore)
		}
	}
...

	metadataEndpoints := api.constructMetadataEndpoints()
	healthEndpoints := api.constructHealthzEndpoints()

	api.endpoints = append(api.endpoints, api.constructStateEndpoints()...)
	api.endpoints = append(api.endpoints, api.constructSecretEndpoints()...)
	api.endpoints = append(api.endpoints, api.constructPubSubEndpoints()...)
	api.endpoints = append(api.endpoints, api.constructActorEndpoints()...)
	api.endpoints = append(api.endpoints, api.constructDirectMessagingEndpoints()...)
	api.endpoints = append(api.endpoints, metadataEndpoints...)
	api.endpoints = append(api.endpoints, api.constructShutdownEndpoints()...)
	api.endpoints = append(api.endpoints, api.constructBindingsEndpoints()...)
	api.endpoints = append(api.endpoints, healthEndpoints...)

	api.publicEndpoints = append(api.publicEndpoints, metadataEndpoints...)
	api.publicEndpoints = append(api.publicEndpoints, healthEndpoints...)

	return api
}

//部分state 端点示例
func (a *api) constructStateEndpoints() []Endpoint {
	return []Endpoint{
		{
			Methods: []string{fasthttp.MethodGet},
			Route:   "state/{storeName}/{key}",
			Version: apiVersionV1,
			Handler: a.onGetState,
		},
...
}

// NewServer returns a new HTTP server.
func NewServer(api API, config ServerConfig, tracingSpec config.TracingSpec, metricSpec config.MetricSpec, pipeline http_middleware.Pipeline, apiSpec config.APISpec) Server {
	return &server{
		api:         api,
		config:      config,
		tracingSpec: tracingSpec,
		metricSpec:  metricSpec,
		pipeline:    pipeline,
		apiSpec:     apiSpec,
	}
}
```
**GRPC Internal Server**

```
msg="enabled gRPC tracing middleware" 
msg="enabled gRPC metrics middleware"
msg="sending workload csr request to sentry"
msg="certificate signed successfully"
msg="internal gRPC server is running on port 50002"
msg="starting workload cert expiry watcher. current cert expires on: 2022-05-13 02:06:10 +0000 UTC"
```


Dapr HTTP API 和 Dapr gRPC API主要用来处理上游服务的接口调用。GRPC Internal Server则主要是用来处理Dapr runtime之间的服务调用。

```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	err = a.startGRPCInternalServer(grpcAPI, a.runtimeConfig.InternalGRPCPort)
	log.Infof("internal gRPC server is running on port %v", a.runtimeConfig.InternalGRPCPort)
	
...
}

func (a *DaprRuntime) startGRPCInternalServer(api grpc.API, port int) error {
	// Since GRPCInteralServer is encrypted & authenticated, it is safe to listen on *
	serverConf := a.getNewServerConfig([]string{""}, port)
	server := grpc.NewInternalServer(api, serverConf, a.globalConfig.Spec.TracingSpec, a.globalConfig.Spec.MetricSpec, a.authenticator, a.proxy)
	if err := server.StartNonBlocking(); err != nil {
		return err
	}
	a.apiClosers = append(a.apiClosers, server)

	return nil
}

// NewInternalServer returns a new gRPC server for Dapr to Dapr communications.
func NewInternalServer(api API, config ServerConfig, tracingSpec config.TracingSpec, metricSpec config.MetricSpec, authenticator auth.Authenticator, proxy messaging.Proxy) Server {
	return &server{
		api:              api,
		config:           config,
		tracingSpec:      tracingSpec,
		metricSpec:       metricSpec,
		authenticator:    authenticator,
		renewMutex:       &sync.Mutex{},
		kind:             internalServer,
		logger:           internalServerLogger,
		maxConnectionAge: getDefaultMaxAgeDuration(),
		proxy:            proxy,
	}
}
```

**阻塞直至Dapr runtime启动**

```
msg="application discovered on port 3000"
```

```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	a.blockUntilAppIsReady()
	
...
}

func (a *DaprRuntime) blockUntilAppIsReady() {
	for {
		conn, _ := net.DialTimeout("tcp", net.JoinHostPort("localhost", fmt.Sprintf("%v", a.runtimeConfig.ApplicationPort)), time.Millisecond*500)
		if conn != nil {
			conn.Close()
			break
		}
		// prevents overwhelming the OS with open connections
		time.Sleep(time.Millisecond * 50)
	}

	log.Infof("application discovered on port %v", a.runtimeConfig.ApplicationPort)
}
```
**创建AppChannel**

```
msg="[DEPRECATION NOTICE] Adding a default content type to incoming service invocation requests is deprecated and will be removed in the future. See https://docs.dapr.io/operations/support/support-preview-features/ for more details. You can opt into the new behavior today by setting the configuration option `ServiceInvocation.NoDefaultContentType` to true." 
```


```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	err = a.createAppChannel()
	a.daprHTTPAPI.SetAppChannel(a.appChannel)
	grpcAPI.SetAppChannel(a.appChannel)
...
}

func (a *DaprRuntime) createAppChannel() error {
	if a.runtimeConfig.ApplicationPort > 0 {
		var channelCreatorFn func(port, maxConcurrency int, spec config.TracingSpec, sslEnabled bool, maxRequestBodySize int, readBufferSize int) (channel.AppChannel, error)

		switch a.runtimeConfig.ApplicationProtocol {
		case GRPCProtocol:
			channelCreatorFn = a.grpc.CreateLocalChannel
		case HTTPProtocol:
			channelCreatorFn = http_channel.CreateLocalChannel
		default:
			return errors.Errorf("cannot create app channel for protocol %s", string(a.runtimeConfig.ApplicationProtocol))
		}

		ch, err := channelCreatorFn(a.runtimeConfig.ApplicationPort, a.runtimeConfig.MaxConcurrency, a.globalConfig.Spec.TracingSpec, a.runtimeConfig.AppSSL, a.runtimeConfig.MaxRequestBodySize, a.runtimeConfig.ReadBufferSize)
		if err != nil {
			log.Infof("app max concurrency set to %v", a.runtimeConfig.MaxConcurrency)
		}

		// TODO: Remove once feature is finalized
		if a.runtimeConfig.ApplicationProtocol == HTTPProtocol && !config.GetNoDefaultContentType() {
			log.Warn("[DEPRECATION NOTICE] Adding a default content type to incoming service invocation requests is deprecated and will be removed in the future. See https://docs.dapr.io/operations/support/support-preview-features/ for more details. You can opt into the new behavior today by setting the configuration option `ServiceInvocation.NoDefaultContentType` to true.")
		}
		a.appChannel = ch
	} else {
		log.Warn("app channel is not initialized. did you make sure to configure an app-port?")
	}

	return nil
}
```
**初始化消息转发API**

```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	a.initDirectMessaging(a.nameResolver)
	a.daprHTTPAPI.SetDirectMessaging(a.directMessaging)
	grpcAPI.SetDirectMessaging(a.directMessaging)
...
}

func (a *DaprRuntime) initDirectMessaging(resolver nr.Resolver) {
	a.directMessaging = messaging.NewDirectMessaging(
		a.runtimeConfig.ID,
		a.namespace,
		a.runtimeConfig.InternalGRPCPort,
		a.runtimeConfig.Mode,
		a.appChannel,
		a.grpc.GetGRPCConnection,
		resolver,
		a.globalConfig.Spec.TracingSpec,
		a.runtimeConfig.MaxRequestBodySize,
		a.proxy,
		a.runtimeConfig.ReadBufferSize,
		a.runtimeConfig.StreamRequestBody,
	)
}
```
direct messaging api在文件`github.com/dapr/dapr/pkg/messaging/direct_messaging.go`的NewDirectMessaging方法中被实现

```
func NewDirectMessaging(
	appID, namespace string,
	port int, mode modes.DaprMode,
	appChannel channel.AppChannel,
	clientConnFn messageClientConnection,
	resolver nr.Resolver,
	tracingSpec config.TracingSpec, maxRequestBodySize int, proxy Proxy, readBufferSize int, streamRequestBody bool,
) DirectMessaging {
	hAddr, _ := utils.GetHostAddress()
	hName, _ := os.Hostname()

	dm := &directMessaging{
		appChannel:          appChannel,
		connectionCreatorFn: clientConnFn,
		appID:               appID,
		mode:                mode,
		grpcPort:            port,
		namespace:           namespace,
		resolver:            resolver,
		tracingSpec:         tracingSpec,
		hostAddress:         hAddr,
		hostName:            hName,
		maxRequestBodySize:  maxRequestBodySize,
		proxy:               proxy,
		readBufferSize:      readBufferSize,
	}

	if proxy != nil {
		proxy.SetRemoteAppFn(dm.getRemoteApp)
		proxy.SetTelemetryFn(dm.setContextSpan)
	}

	return dm
}

```

**开始监听订阅**

```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	a.startSubscribing()
...
}

func (a *DaprRuntime) startSubscribing() {
	for name, pubsub := range a.pubSubs {
		if err := a.beginPubSub(name, pubsub); err != nil {
			log.Errorf("error occurred while beginning pubsub %s: %s", name, err)
		}
	}
}
```

**开始监听绑定**


```
func (a *DaprRuntime) initRuntime(opts *runtimeOpts) error {
...
	a.startReadingFromBindings()
...
}

func (a *DaprRuntime) startReadingFromBindings() error {
	if a.appChannel == nil {
		return errors.New("app channel not initialized")
	}
	for name, binding := range a.inputBindings {
		go func(name string, binding bindings.InputBinding) {
			if !a.isAppSubscribedToBinding(name) {
				log.Infof("app has not subscribed to binding %s.", name)
				return
			}

			err := a.readFromBinding(name, binding)
			if err != nil {
				log.Errorf("error reading from input binding %s: %s", name, err)
			}
		}(name, binding)
	}
	return nil
}
```

**Ending**

```
msg="dapr initialized. Status: Running. Init Elapsed 3243.280519ms"
```
Dapr runtime初始化启动完成！
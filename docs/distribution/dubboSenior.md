#### 服务注册过程分析

ServiceConfig类：

- final Protocol PROTOCOL
- final ProxyFactory PROXY_FACTORY
- protected T ref：继承于父类ServiceConfigBase

具体服务到invoker的转换：

```java
Invoker<?> invoker = PROXY_FACTORY.getInvoker(ref, (Class) interfaceClass, 
    registryURL.addParameterAndEncoded(EXPORT_KEY, url.toFullString()));
DelegateProviderMetaDataInvoker wrapperInvoker = new 
    DelegateProviderMetaDataInvoker(invoker, this);
```

invoker转换成Exporter过程：

- RegistryService：对指定路径进行注册，解绑，监听和取消监听，查询操作

```java
public interface RegistryService {
  /**
  * 进行对URL的注册操作，比如provider，consumer，routers等
  */
  void register(URL url);
  /**
  * 解除对指定URL的注册，比如provider，consumer，routers等
  */
  void unregister(URL url);
  /**
  * 增加对指定URL的路径监听，当有变化的时候进行通知操作
  */
  void subscribe(URL url, NotifyListener listener);
  /**
  * 解除对指定URL的路径监听，取消指定的listener
  */
  void unsubscribe(URL url, NotifyListener listener);
  /**
  * 查询指定URL下面的URL列表，比如查询指定服务下面的consumer列表
  */
  List<URL> lookup(URL url);
}
```

- RegistryFactory：生成真实的注册中心。可以保证一个应用可以使用多个注册中心，通过不同的protocol参数，来选择不同的协议

```java
@SPI("dubbo")
public interface RegistryFactory {
  /**
  * 获取注册中心地址
  */
  @Adaptive({"protocol"})
  Registry getRegistry(URL url);
}
```

- RegistryProtocol类：管理整个注册中心相关协议，并且统一对外提供服务

```java
// 将我们需要执行的信息注册并且导出
public <T> Exporter<T> export(final Invoker<T> originInvoker) throws
RpcException {
  	// 获取注册中心的地址
    URL registryUrl = getRegistryUrl(originInvoker);
    // 获取当前提供者需要注册的地址
    URL providerUrl = getProviderUrl(originInvoker);
    // 获取进行注册override协议的访问地址
    final URL overrideSubscribeUrl = getSubscribedOverrideUrl(providerUrl);
  	// 增加override的监听器
  	final OverrideListener overrideSubscribeListener = new
		OverrideListener(overrideSubscribeUrl, originInvoker);
  	overrideListeners.put(overrideSubscribeUrl, overrideSubscribeListener);
    // 根据现有的override协议，对注册地址进行改写操作
  	providerUrl = overrideUrlWithConfig(providerUrl, overrideSubscribeListener);
  	// 对当前的服务进行本地导出
  	// 完成后即可在看到本地的20880端口号已经启动，并且暴露服务
  	final ExporterChangeableWrapper<T> exporter = doLocalExport(originInvoker,
providerUrl);
    
  	// 获取真实的注册中心, 比如我们常用的ZookeeperRegistry
  	final Registry registry = getRegistry(originInvoker);
    // 获取当前服务需要注册到注册中心的providerURL，主要用于去除一些没有必要的参数(比如在本
			//地导出时所使用的qos参数等值)
    final URL registeredProviderUrl = getUrlToRegistry(providerUrl,registryUrl);
    // 获取当前url是否需要进行注册参数
  	boolean register = providerUrl.getParameter(REGISTER_KEY, true);
  	if (register) {
    	// 将当前的提供者注册到注册中心上去
    	register(registryUrl, registeredProviderUrl);
 	}
  	// 对override协议进行注册，用于在接收到override请求时做适配,这种方式用于适配2.6.x及之
		// 前的版本(混用)
  	registry.subscribe(overrideSubscribeUrl, overrideSubscribeListener);
  	// 设置当前导出中的相关信息
  	exporter.setRegisterUrl(registeredProviderUrl);
    exporter.setSubscribeUrl(overrideSubscribeUrl);
  	// 返回导出对象(对数据进行封装)
  	return new DestroyableExporter<>(exporter);
}
```







dubbo:service和dubbo:reference

mock：consumer端  reference中属性设置为true，当超时时间到后，调用本地提供的*Mock类进行处理

timeout: 以consumer端为准， dubbo:consumer 中timeou属性为主，没有该属性时，以reference中timeout属性为主，provider端中的timeout为辅





服务远程运维命令：telnet ip host   如：telnet localhost 22222 查看dubbo服务，ls 查看列表


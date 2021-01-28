# Dubbo

### 官方文档随笔
* Dubbo缺省的负载均衡是Random随机均衡，此外还支持RoundRobin、LeastActive、ConsistentHash。一致性哈希（相同参数的请求总是发到同一提供者）是常用重点
* Dubbo的线程模型提供了五种线程池分发策略和四种线程池。http://dubbo.apache.org/zh/docs/v2.7/user/examples/thread-model/
* Dubbo中可以配置多协议：如大数据用短连接协议，小数据大并发用长连接协议（这里为什么这样推荐）
* Dubbo可以通过Protobuf编码，通过IDL（Interface Description Language 接口描述语言）定义服务，实现跨语言调用

* Dubbo的所有RPC接口都实现了回声测试EchoService，只需要将接口强转为EchoService即可使用回声测试
* Dubbo提供了基于java.util.concurrent.CompletableFuture\<T\>的异步调用和异步执行，使用者可以正交组装，得到四种方式 
*  Dubbo还提供了“injvm”作为本地调用，不开启网络端口，但会流经Filter链
*  Dubbo并发控制可以通过配置executes参数来限制线程池大小，粒度有interface和method。也可以通过actives配置每个客户端并发执行或者占用连接的请求数。如果service和refrence都配置了actives，以reference优先。 另外还可以通过将interface的loadbalance策略配置为leastactive，此 Loadbalance 会调用并发数最小的 Provider（Consumer端并发数）。
* Dubbo可以通过配置路由规则实现读写分离等服务过滤，粒度有application和service
* Dubbo可以通过配置mock来做到服务降级（但貌似不太智能，只能手动配置？？？）
* Dubbo在服务侧线程池优化，限制了消费端突然出现大量线程导致OOM
* Dubbo使用JDK的ShutdownHook 实现优雅停机，并存在停机超时

### Dubbo源码实现

#### SPI 
SPI源于Java SPI，是通过配置来实现动态为接口替换实现类的过程，比如JDBC的Driver配置就是通过Java SPI实现的。 
Dubbo扩展了Dubbo SPI，可以动态配置所需的扩展类。比如Protocal、LoadBalance的配置都是通过SPI进行组装的。  
**源码分析待补齐**
#### 服务导出
Dubbo 服务导出过程始于 Spring 容器发布刷新事件，Dubbo 在接收到事件后，会立即执行服务导出逻辑。
1. 检查配置的合法性（SeviceConfig.export()），主要是检查interface和一些subconfig,并且检查是否配置了export和delay
2. 接下来在doExportUrls中对多协议多注册中心导出提供了服务
3. 配置检查结束之后就通过加载进来的配置组装URL。
4. 最后完善一下Url，判断一下local还是remote，构建Invoker
5. 构建Invoker（JavassistProxyFactory中，用Wrapper通过反射包装，然后利用AbstractProxyInvoker代理wrapper，这里wrapper使用了javassist）
6. 导出服务到本地会在运行时调用 InjvmProtocol 的 export 方法，逻辑并不复杂。接下来是导出服务到远程，主要做以下几件事  
	* 调用 doLocalExport 导出服务  
	* 向注册中心注册服务  
	* 向注册中心进行订阅 override 数据  
	* 创建并返回 DestroyableExporter  
	i. 调用doLocalExport主要是构建Server，默认是NettyServer，并bind到网络端口
	ii. 服务注册以Zookeeper为例，创建了注册中心（AbstractRegistryFactory ），创建ZookeeperTransporter（自适应SPI，默认CuratorZookeeperTransporter ）。这里用到的是Curator框架（Zookeeper客户端框架），可以单独了解。  
	以 Zookeeper 为例，所谓的服务注册，本质上是将服务配置数据写入到 Zookeeper 的某个路径的节点下。进入FailbackRegister以及其子类ZookeeperRegister中发现无非也就是如此。  
	总结这部分就是先创建注册中心实例，之后再通过注册中心实例注册服务。
	iii. 向注册中心进行订阅 override 数据  (待补充)
	iv.  创建并返回 DestroyableExporter   (待补充) 
	

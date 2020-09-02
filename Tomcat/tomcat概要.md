# Tomcat概要
> 《How Tomcat Works》 《深入剖析Tomcat》 《Tomcat架构解析》  
> Tomcat是Apache开源的Web服务器，其以Catalina作为Servlet容器，采用Pipeline作为服务的连续触发，Lifecycle管理所有组件的生命周期。  
![Tomcat总体架构](https://github.com/Joey777210/My_Note/blob/master/Pics/tomcat1.jpg)
对Tomcat的运行机制很感兴趣，凭着新手的理解，结合上面的书籍浅读了一下Tomcat源码下面进行一些个人理解的总结。
# Tocmat中什么最重要
## Tomcat架构的简单描述
    要理解Tomcat，就要理解Http，一次Http请求的过程中发生了什么，将要发生什么，可能会发生什么。    
    跟着上面那张图，把Tomcat的基本架构理顺一下。首先，一个简单的服务器可以被描述为Server，它有启动和停止命令，能够使用Socket监听端口；我们的服务器需要处理连接和处理请求，将它们解耦，我们有了Connector和Container，其中Connector负责开启Socket并监听客户端请求、返回响应数据；Container负责处理请求。  
    在上面的模式中，如果我们需要让他们形成工作流，一个Server是无法管理多个Connector和Container的对应的，所以多了一层Service，用来管理多个Connector和一个Container的对应，而Server管理多个Service。
    接下来把Container命名为Engine，表示整个Servlet引擎。  
    一个Engine下面可以有多个应用，一个应用为一个Context实际上对应的就是一个Web Application，我们可以在配置文件中修改端口配置多个Context，也就实现了一个Tomcat中运行多个项目的目的。  
    Host代表一个虚拟域名，内部嵌套Context。  
    一个Wrapper对应一个Servlet，Context中有多个Wrapper。  
    我们再次提起Container，这里以上提到的Engine，Host，Context，Wrapper均继承自Container。  
    再抽象一层，我们需要管理所有组件，包括Server和Service的生命周期，我们就让所有的组件都实现一个管理生命周期的接口，称之为LifeCycle。  
    从Engine到Host，再到Context，再到Wrapper，都维护了一个Pipeline，有基础Valve，其层级之间通过PipeLline连接，并可以通过Valve对请求处理进行扩展（比如密码校验等）。这里叫做责任链模式。  
    接下来我们再看Connector的设计，连接器完成的任务有  
        1，监听端口，读取请求  
        2，将请求数据按照指定协议解析  
        3，根据请求地址匹配正确的容器进行处理  
        4，容器处理完成后，将响应返回客户端  
    从这里我们可以看到，连接其需要判断不同的协议，还要选择不同的容器。这里Tomcat对其进行了解耦，设计了Endpoint作为IO，监听请求；Processor按照请求地址映射到容器进行处理（Mapper）。Tomcat通过适配器模式实现了Connector与Mapper、Container的解耦，默认为CoyoteAdapter。  
    为了解决并发问题，Tomcat设计了Executor作为可在组件间共享的线程池，实现了Lifecycle，可按照通用组件进行管理。Executor由Service维护，所以共享范围为同一个Service。  
    到这里Tomcat的基本结构就差不多了。  
## Tomcat初始化流程  
![Tomcat初始化流程](https://github.com/Joey777210/My_Note/blob/master/Pics/tomcat2.jpg)
    Tomcat的主类在Bootstrap，初始化后进入Catalina类，在Catalina的load方法中的digester.parse(inputSource)中解析了配置文件，并初始化了Service（包括初始化了Connector、Endpoint等）。  
    注意，各种组件的初始化都是由从Lifecycle继承来的initInternal方法在做，且都要在函数中调用super.initInterval()(实际上是LifecycleMBeanBase类中的此方法)，用来注册Bean。  
    在Catalina中getServer并初始化和启动了它。这里StandardServer主要用来监听Stop命令，真正的请求监听还是Endpoint在做。  
## 一次请求是如何被接收、处理、返回的，流程如何
![Tomcat初始化流程](https://github.com/Joey777210/My_Note/blob/master/Pics/tomcat3.jpg)
## 如何处理并发请求的  

## Servlet的原理、加载机制
## Session和Cookie是怎么生成、使用、存储的
## 过滤器Filter是如何加载的
## 热加载
## Tomcat中的设计模式
## NIO AIO BIO
## 
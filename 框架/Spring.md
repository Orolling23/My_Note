# Spring
## 什么是Spring框架
Spirng是一个旨在提高开发效率，以及系统可维护性的开发框架。  
我们一般说 Spring 框架指的都是 Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是：**核心容器、数据访问/集成,、Web、AOP（面向切面编程）、工具、消息和测试模块**。比如：**Core Container 中的 Core 组件是Spring 所有组件的核心，Beans 组件和 Context 组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。**  
Spring 官网列出的 Spring 的 6 个特征:  
* 核心技术 ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
* 测试 ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
* 数据访问 ：事务，DAO支持，JDBC，ORM，编组XML。
* Web支持 : Spring MVC和Spring WebFlux Web框架。
* 集成 ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
* 语言 ：Kotlin，Groovy，动态语言。
## Spring的重要模块
* Spring Core： 基础,可以说 Spring 其他所有的功能都需要依赖于该类库。主要提供 IoC 依赖注入功能。
* Spring Aspects ： 该模块为与AspectJ的集成提供支持。
* Spring AOP ：提供了面向切面的编程实现。
* Spring JDBC : Java数据库连接。
* Spring JMS ：Java消息服务。
* Spring ORM : 用于支持Hibernate等ORM工具。
* Spring Web : 为创建Web应用程序提供支持。
* Spring Test : 提供了对 JUnit 和 TestNG 测试的支持。
## IoC
IoC(Inverse of Control：控制反转)是一种设计思想，就是将**原本在程序中手动创建对象的控制权，交由Spring框架管理。IoC容器是Spring实现IoC的载体，IoC容器实际上是个Map，存放的是各种对象**  
将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。 **IoC 容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。** 在实际项目中一个 Service 类可能有几百甚至上千个类作为它的底层，假如我们需要实例化这个 Service，你可能要每次都要搞清这个 Service 所有底层类的构造函数，这可能会把人逼疯。如果利用 IoC 的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。
### Spring IoC的初始化过程
XML —读取—> Resource —解析—> BeanDefinition —注册—> BeanFactory
### 参考文章
https://www.zhihu.com/question/23277575/answer/169698662  
IoC源码 https://javadoop.com/post/spring-ioc  
https://www.zhihu.com/question/313785621  
## AOP
AOP(Aspect-Oriented Programming：面向切面编程)能够将哪些与业务无关，**却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。**  
Spring AOP就是**基于动态代理**的，**如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象**，**而对于没有实现接口的对象，就无法使用 JDK Proxy 去进行代理了，这时候Spring AOP会使用Cglib ，这时候Spring AOP会使用 Cglib 生成一个被代理对象的子类来作为代理**  
</br>
另外也可以集成AspectJ实现AOP
### Spring AOP和AspectJ AOP的区别
Spring AOP 属于**运行时增强**，而 AspectJ 是**编译时增强**。 Spring AOP 基于**代理(Proxying)**，而 AspectJ 基于**字节码操作(Bytecode Manipulation)**。  
## Spring Bean
### bean 作用域有哪些
有以下几种作用域：  
* singleton : 唯一 bean 实例，Spring 中的 bean **默认**都是单例的。
* prototype：每次请求都会创建一个新的bean实例
* request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效
* session：每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效。
* global-session： 全局session作用域，仅仅在基于portlet的web应用中才有意义，Spring5已经没有了。Portlet是能够生成语义代码(例如：HTML)片段的小型Java Web插件。它们基于portlet容器，可以像servlet一样处理HTTP请求。但是，与 servlet 不同，每个 portlet 都有不同的会话
### Spring中 单例bean的线程安全问题
**存在安全问题。因为，当多个线程操作同一个对象的时候，对这个对象的成员变量的写操作会存在线程安全问题。**  
但是，一般情况下，我们常用的 Controller、Service、Dao 这些 Bean 是**无状态的。无状态的 Bean 不能保存数据，因此是线程安全的。**  
解决办法：
1. 在类中使用ThreadLocal，将需要的可变变量都保存在ThreadLocal中（推荐）  
2. 改变Bean作用域为prototype，这样每次请求都会创建一个新的bean，且仅在当前请求中有效，也就避免了线程安全问题  
### @Component和@Bean的区别
1. 作用对象不同: @Component 注解作用于类，而@Bean注解作用于方法。
2. @Component通常是**通过类路径扫描来自动侦测以及自动装配到Spring容器中**（我们可以使用 @ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。@Bean 注解通常是我们**在标有该注解的方法中定义产生这个 bean**,@Bean告诉了Spring这是某个类的示例，当我需要用它的时候还给我。
3. @Bean 注解比 Component 注解的自定义性更强，而且很多地方我们只能通过 @Bean 注解来注册bean。比如当我们引用第三方库中的类需要装配到 Spring容器时，则只能通过 @Bean来实现。一般和@Configuration搭配使用  
示例：
```
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }

}
```
上面的代码相当于xml配置
```
<beans>
    <bean id="transferService" class="com.acme.TransferServiceImpl"/>
</beans>
```
### bean生命周期
1. Bean容器找到配置文件中Spring Bean的定义
2. Bean容器利用Java反射 API创建一个Bean实例（实例化）
3. 如果涉及到属性值，利用set()方法设置
4. 如果Bean实现了`BeanNameAware`接口，调用`setBeanName()`方法，设置Bean名字
5. 如果 Bean 实现了 `BeanClassLoaderAware` 接口，调用 `setBeanClassLoader()`方法，传入 ClassLoader对象的实例。
6. 与上面的类似，如果实现了其他 \*.Aware接口，就调用相应的方法。
7. 如果有和加载这个 Bean 的 Spring 容器相关的 `BeanPostProcessor` 对象，执行`postProcessBeforeInitialization() `方法
8. 如果Bean实现了`InitializingBean`接口，执行`afterPropertiesSet()`方法。
9. 如果 Bean 在配置文件中的定义包含` init-method `属性，执行指定的方法。
10. 如果有和加载这个 Bean的 Spring 容器相关的` BeanPostProcessor `对象，执行`postProcessAfterInitialization() `方法
11. 当要销毁 Bean 的时候，如果 Bean 实现了` DisposableBean `接口，执行` destroy() `方法。
12. 当要销毁 Bean 的时候，如果 Bean 在配置文件中的定义包含` destroy-method` 属性，执行指定的方法。

上面的过程看起来非常复杂，我也不清楚面试是不是要这样回答。下面尽量尝试从原理去理解  
**以下重点理解**
**这篇博客重点看** https://www.jianshu.com/p/1dec08d290c1  
https://www.cnblogs.com/javazhiyin/p/10905294.html  
Spring Bean生命周期分为四个部分：
1. 实例化
2. 属性赋值
3. 初始化
4. 销毁
实际上对应的就是上面的1，2，3，9，12，而其他的几个步骤，就是通过实现Spring提供的接口，而影响Bean生命周期。分为两类接口  
#### 第一类：影响多个Bean的接口
实现了这些接口的Bean会切入到多个Bean的生命周期中。正因为如此，这些接口的功能非常强大，Spring内部扩展也经常使用这些接口，例如自动注入以及AOP的实现都和他们有关。
* BeanPostProcessor
* InstantiationAwareBeanPostProcessor
**这两个接口是Spring扩展中最重要的两个接口**。**InstantiationAwareBeanPostProcessor作用于实例化阶段的前后，BeanPostProcessor作用于初始化阶段的前后。**正好和第一、第三个生命周期阶段对应。通过图能更好理解：  
![生命周期](./Pics/bean生命周期.jpg)  
#### 第二类：只调用一次的接口
这一类接口功能丰富，常用于用户自定义扩展，其中又可以分为两类：
1. Aware类型的接口
2. 生命周期接口
* Aware接口的作用就是让我们能够拿到Spring容器中的一些资源，例如BeanNameAware可以拿到BeanName，以此类推。调用时机需要注意：**所有的Aware方法都是在初始化阶段之前调用的！**
* 生命周期接口：两个生命周期接口`InitializingBean`和`DisposableBean`,用来自己实现bean初始化和销毁两个生命周期
* 

## Spring MVC
* Model1时代，JSP又作表现层又作控制层，少量Java bean来访问数据库，导致控制逻辑和表现逻辑混杂，代码重用率极低；前后端相互依赖，开发效率低，难以测试
* Model2时代，Java Bean + JSP + Servlet，早期的MVC，已经开始有拆解分层的趋势，但还有很多问题。抽象和封装程度不够，代码难以重用，后来出现了Spring MVC。  
* Spring MVC 下我们一般把后端项目分为 Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层(控制层，返回数据给前台页面)。
### Spring MVC工作原理
![MVC](./Pics/SpringMVC.jpg)  
流程：
1. 客户端发送请求，直接请求到`DispatcherServlet`
2. `DispatcherServlet`根据请求信息调用 `HandlerMapping`，解析请求对应的 `Handler`(也就是我们说的`Controller`控制器)。
3. 解析到对应的` Handler`（也就是我们平常说的 Controller 控制器）后，开始由 `HandlerAdapter` 适配器处理。
4. `HandlerAdapter` 会根据` Handler `来调用真正的处理器来处理请求，并处理相应的业务逻辑。
5. 处理器处理完业务后，会返回一个 `ModelAndView `对象，`Model` 是返回的数据对象，`View` 是个逻辑上的 View。
6. `ViewResolver` 会根据逻辑 `View` 查找实际的 `View`。
7. `DispaterServlet` 把返回的` Model` 传给` View`（视图渲染）。
8. 把 `View` 返回给请求者（浏览器）
## Spring框架中用到的设计模式
https://mp.weixin.qq.com/s?__biz=Mzg2OTA0Njk0OA==&mid=2247485303&idx=1&sn=9e4626a1e3f001f9b0d84a6fa0cff04a&chksm=cea248bcf9d5c1aaf48b67cc52bac74eb29d6037848d6cf213b0e5466f2d1fda970db700ba41&token=255050878&lang=zh_CN#rd  
* 工厂设计模式 : Spring使用工厂模式通过 BeanFactory、ApplicationContext 创建 bean 对象。
* 代理设计模式 : Spring AOP 功能的实现。
* 单例设计模式 : Spring 中的 Bean 默认都是单例的。
* 模板方法模式 : Spring 中 jdbcTemplate、hibernateTemplate 等以 Template 结尾的对数据库操作的类，它们就使用到了模板模式。
* 包装器设计模式 : 我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。
* 观察者模式: Spring 事件驱动模型就是观察者模式很经典的一个应用。
* 适配器模式 :Spring AOP 的增强或通知(Advice)使用到了适配器模式、spring MVC 中也是用到了适配器模式适配Controller。
* ......
## Spring 事务
### Spring管理事务的方式有几种？
1. 编程式事务，在代码中硬编码（不推荐）
2. 声明式事务，在配置文件中配置（推荐）
声明式事务又分为基于XML的和基于注解的声明式事务  
### Spring事务中五种隔离级别
* TransactionDefinition.ISOLATION_DEFAULT: 使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
* TransactionDefinition.ISOLATION_READ_UNCOMMITTED: 最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读
* TransactionDefinition.ISOLATION_READ_COMMITTED: 允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生
* TransactionDefinition.ISOLATION_REPEATABLE_READ: 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
* TransactionDefinition.ISOLATION_SERIALIZABLE: 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。
### Spring事务中七种事务传播行为
**支持当前事务的情况**
* TransactionDefinition.PROPAGATION_REQUIRED： 如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。
* TransactionDefinition.PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。
* TransactionDefinition.PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）
**不支持当前事务的情况：**
* TransactionDefinition.PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。
* TransactionDefinition.PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。
* TransactionDefinition.PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。
**其他情况：**
* TransactionDefinition.PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。
### @Transactional(rollbackFor = Exception.class)注解
我们知道：Exception分为运行时异常RuntimeException和非运行时异常。事务管理对于企业应用来说是至关重要的，即使出现异常情况，它也可以保证数据的一致性。  
**当@Transactional注解作用于类上时，该类的所有 public 方法将都具有该类型的事务属性，同时，我们也可以在方法级别使用该标注来覆盖类级别的定义。如果类或者方法加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。**  
在@Transactional注解中如果**不配置rollbackFor属性,那么事务只会在遇到RuntimeException的时候才会回滚,加上rollbackFor=Exception.class,可以让事务在遇到非运行时异常时也回滚。**  
https://developer.ibm.com/zh/articles/j-master-spring-transactional-use/








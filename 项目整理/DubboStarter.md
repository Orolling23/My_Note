## DubboStarter
定义了一个DubboAutoConfiguration，用来向Dubbo中注册Appllo上的配置，提供公司内部的starter框架方法。
#### @Configuration注解
@Configuration用于定义**配置类**，可替换xml配置文件，被注解的类内部包含有一个或多个被@Bean注解的方法，这些方法将会被AnnotationConfigApplicationContext或AnnotationConfigWebApplicationContext类进行扫描，并用于构建bean定义，初始化Spring容器。  
#### @ConditionalOnClass
@ConditionalOnClass(value={RegistryService.class})
当给定的类名在类路径上存在，则实例化当前Bean
#### @AutoConfigureAfter
@AutoConfigureAfter(value=ApolloAutoConfiguration.class)
在加载配置的类之后再加载当前类
#### @ImportResource
@ImportResource(value={"classpath*:dubbo/**/*.xml"})
#### @@EnableApolloConfig
@EnableApolloConfig(ApolloConstants.DEFAULT_DUBBO_NAMESPACE)
#### @Import
@Import(DubboSupportsConfiguration.class)
#### config
```
public class DubboAutoConfiguration {
	
	@Bean
	public DubboAutoConfigurationRegistrar dubboAutoConfigurationRegistar(){
		return new DubboAutoConfigurationRegistrar();
	}
}

```
#### 配置类做了什么
DubboAutoConfigurationRegistrar这个类作为配置类，实现BeanDefinitionRegistryPostProcessor这个Spring接口。  
主要是复写了这个`postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry)`方法.在里面构造了DubboBeanDefinitionBuilder，在它的build方法中构造了RootBeanDefinition，并将其注册在BeanDefinitionRegistry中。  
RootBeanDefinition中的各类参数是通过Dubbo的各种Config class的反射获取其属性，然后将Apollo上的参数取出来对应配置进去。  
对应配置的config类别有ApplicationConfigBeanDefinition,Protocal,Registry,Provider,Consumer,Annotation,Monitor，Apollo上的属性路径是写死的
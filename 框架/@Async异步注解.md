# @Async异步注解
## 背景
调用风控引擎，计算时间长，同步调用的话整个调用链路时间会很长。所以需要异步调用。  

而引擎由于那次迭代没有时间将同步接口修改为异步接口，所以在调用端利用@Async注解来异步化。    

## @Async是什么
在Spring中，基于@Async标注的方法，叫做异步方法。**这些方法将在执行的时候，将会在独立的线程中被执行，调用者无需等待它的完成，即可继续其他的操作。**   

@Async异步方法分为无返回值的和有返回值的。   

## 怎么用
### 启用
1. 方法一：基于Java配置的启用方式  

```
@Configuration
@EnableAsync
public class SpringAsyncConfig { ... }
```
2. 方法二：基于XML配置的启用方式  

```
<task:executor id="myexecutor" pool-size="5"  />
<task:annotation-driven executor="myexecutor"/>
```
### 使用
在异步处理的方法上添加注解，然后调用即可达到效果。  

**注意调用方和被调用方不能在一个类中**，原因：这是由于 **Spring AOP (包括动态代理和 CGLIB 的 AOP) 的限制**导致的. Spring AOP 并不是扩展了一个类(目标对象), 而是使用了一个**代理对象来包装目标对象, 并拦截目标对象的方法调用**. 这样的实现带来的影响是: **在目标对象中调用自己类内部实现的方法时, 这些调用并不会转发到代理对象中, 甚至代理对象都不知道有此调用的存在**.  

```
@Component
@Slf4j
public class AsyncTask {

    @Async
    public void dealNoReturnTask(){
        log.info("Thread {} deal No Return Task start", Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        log.info("Thread {} deal No Return Task end at {}", Thread.currentThread().getName(), System.currentTimeMillis());
    }
}
```

如果异步调用要返回数据，就在定义方法时，**采用Future接口的返回值，具体的实现类型是AsyncResult**，返回数据类型可以泛型约束。  

```
    @Async
    public Future<String> dealHaveReturnTask() {
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("thread", Thread.currentThread().getName());
        jsonObject.put("time", System.currentTimeMillis());
        return new AsyncResult<String>(jsonObject.toJSONString());
    }
```

可以调用`Future.isCancelled()`和`Future.isDone()`判断异步任务的状态  

### 自定义线程池
自定义线程池可以实现**自定义异常处理、自定义日志机制**等功能。  

1. 自定义实现AsyncTaskExecutor的任务执行器
在这里定义处理具体异常的逻辑和方式
2. 配置由自定义的TaskExecutor替代内置的任务执行器

```
public class ExceptionHandlingAsyncTaskExecutor implements AsyncTaskExecutor {
    private AsyncTaskExecutor executor;
    public ExceptionHandlingAsyncTaskExecutor(AsyncTaskExecutor executor) {
        this.executor = executor;
     }
      ////用独立的线程来包装，@Async其本质就是如此
    public void execute(Runnable task) {
      executor.execute(createWrappedRunnable(task));
    }
    public void execute(Runnable task, long startTimeout) {
        /用独立的线程来包装，@Async其本质就是如此
       executor.execute(createWrappedRunnable(task), startTimeout);
    }
    public Future submit(Runnable task) { return executor.submit(createWrappedRunnable(task));
       //用独立的线程来包装，@Async其本质就是如此。
    }
    public Future submit(final Callable task) {
      //用独立的线程来包装，@Async其本质就是如此。
       return executor.submit(createCallable(task));
    }

    private Callable createCallable(final Callable task) {
        return new Callable() {
            public T call() throws Exception {
                 try {
                     return task.call();
                 } catch (Exception ex) {
                     handle(ex);
                     throw ex;
                   }
                 }
        };
    }

    private Runnable createWrappedRunnable(final Runnable task) {
         return new Runnable() {
             public void run() {
                 try {
                     task.run();
                  } catch (Exception ex) {
                     handle(ex);
                   }
            }
        };
    }
    private void handle(Exception ex) {
      //具体的异常逻辑处理的地方
      System.err.println("Error during @Async execution: " + ex);
    }
}
```

配置文件替代默认的线程池  

```
<task:annotation-driven executor="exceptionHandlingTaskExecutor" scheduler="defaultTaskScheduler" />
<bean id="exceptionHandlingTaskExecutor" class="nl.jborsje.blog.examples.ExceptionHandlingAsyncTaskExecutor">
    <constructor-arg ref="defaultTaskExecutor" />
</bean>
<task:executor id="defaultTaskExecutor" pool-size="5" />
<task:scheduler id="defaultTaskScheduler" pool-size="1" />
```

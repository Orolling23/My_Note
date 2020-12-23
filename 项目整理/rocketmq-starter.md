###  rocketmq-starter
#### 注册consumer
定义一个MQMessageListener  
```
public class LoanRiskTaskTimingReplayListener extends MQMessageListener<String> implements MessageListener {

    @Resource
    LoanRiskTaskTimingReplayTask loanRiskTaskTimingReplayTask;

    @Override
    protected ConsumeConcurrentlyStatus onMessageReceive(String message, Map<String, String> msgProperties) {
        log.info("LoanRiskTaskTimingReplayListener - Receive data from rocketmq, msg={}, messageProp={}", message, msgProperties);
        try {
            loanRiskTaskTimingReplayTask.invoke(message);
        } catch (RetryException ex) {
            log.error("Consume consumeMessage error, but need retry! msg=" + message, ex);
            return ConsumeConcurrentlyStatus.RECONSUME_LATER;
        } catch (Exception e) {
            log.error("Consume consumeMessage error, msg=" + message, e);
        }
        return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
    }
}
```
然后把这个Listener通过MQConfig注册进去。  
```
public class MqConfig extends MQListenerRegistry {

    @Resource
    LoanRiskServiceTopics loanRiskServiceTopics;

    @Override
    public void registerListeners(List<MQConsumerSpec> consumerSpeces) {
        consumerSpeces.add(MQConsumerSpecBuilders.build(loanRiskServiceTopics.getRiskTaskTimingReplayHistOrderTopic(),
                "", LoanRiskTaskTimingReplayListener.class, 1));
    }
}
```
### register做了什么
register所作的事情还是比较简单，因为这里的Listener我是通过注册的方式注册进去的，所以直接进入了他里面的List，但他register里面还要扫描一下通过注解注册的Listener，也加到List里面。接下来就是把这个consumerList中的consumer一个一个通过RocketMQ原生的接口定义成DefaultMQPushConsumer然后调用他的start方法开始监听。
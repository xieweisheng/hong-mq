参考链接:https://www.imooc.com/article/49814

#RabbitMQ实现消息可靠性投递
RabbitMQ可视化监控平台 http://localhost:15672

1. 采用微服务架构，将消息服务本身作为一个微服务注册到Eureka上，
   这样在消息服务回调通知业务方时，可以采取随机选取的负载均衡策略，
   调用服务方相应的回调接口。
   
2. 约定：如果不同的服务需要隔离各自的消息，除了可以设置不同的virtual-host进行粗粒度的隔离外，
   还可以约定将不同服务对应的exchange命名为{service-id}-exchange的格式进行细粒度的隔离。
   如果是通用的exchange，可以使用com.hong.constant.MessageConstants里的ExchangeConstants类命名。
   
3. Caused by: com.rabbitmq.client.ShutdownSignalException: connection error; protocol method: #method<connection.close>(reply-code=530, reply-text=NOT_ALLOWED - access to vhost '/' refused for user 'mq', class-id=10, method-id=40)
   原因：服务器该用户没有分配权限
   解决步骤：
    (1)查看所有用户
      D:\RabbitMQ\RabbitMQ_Server3.7.7\rabbitmq_server-3.7.7\sbin
      λ rabbitmqctl.bat list_users
      Listing users ...
      guest   [administrator]
      root    [administrator]
    (2)如果没有添加用户，则添加用户，rabbitmq-server默认只有一个Guest用户    
      rabbitmqctl add_user username password
    (3)删除用户
      rabbitmqctl delete_user guest
    (4)授权
       λ rabbitmqctl set_permissions -p "/" root ".*" ".*" ".*"
       Setting permissions for user "root" in vhost "/" ...
       
4. RabbitMQ之延迟队列(延迟消费+延迟重试)
   RabbitMQ的两个特性，一个是Time-To-Live Extensions，另一个是Dead Letter Exchanges       
   https://www.cnblogs.com/xishuai/p/spring-boot-rabbitmq-delay-queue.html
   https://blog.csdn.net/zhangyuxuan2/article/details/82986802
   
5.RabbitMQ如何实现高可靠性,即保证生产数据不丢失?
分以下情况进行讨论:
(1)线上服务宕机时,如消费者服务，刚收到了一个订单消息，但是在完成消息的处理之前，也就是还没对订单完成仓储调度发货，结果这个仓储服务突然就宕机了，这个时候会发生什么事情？
   RabbitMQ的自动ACK机制:默认情况下,RabbitMQ在投递完一条消息后就自动确认这条消息消费完毕了,然后把这条消息标记为删除.
   channel.basicConsume(QUEUE_NAME,Boolean.FALSE,deliverCallback,consumerTag -> {});
   == Boolean.FALSE:关闭RabbitMQ默认的AutoAck
   
   
   
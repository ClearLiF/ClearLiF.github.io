---
layout: post
title:  "ActiveMQ的简单配置"
date:   2020-4-22
categories: 消息中间件
tags: ActiveMQ
author: Clear Li
---



在客户端与服务器进行通讯时.客户端调用后，必须等待服务对象完成处理返回结果才能继续执行。

 **客户与服务器对象的生命周期紧密耦合,客户进程和服务对象进程都都必须正常运行;如果由于服务对象崩溃或者网络故障导致用户的请求不可达,客户会受到异常**

**点对点通信:** **客户的一次调用只发送给某个单独的目标对象。**











### 什么是消息中间件

消息中间件是面向消息的，发送者发送给消息服务器，消息服务器将消息存在队列中，这种模式下消息的发送和接收是异步的。例如订单的确定，在前台确定订单后，无需等待后台的管理人员进行确认。

### JMS

JMS是Java的消息服务，JSM的客户端之间是可以通过JMS服务进行异步的消息传输。

有两种消息模型

#### p2p

点对点消息模型，确定每一个消息必须只有一个消费者，是一对一的关系。如果希望每一个消息都要被成功处理的话，应该使用P2P模式。类似于微信发消息的一种模式

#### Pub/Sub订阅

与点对点消息模型不一样的一点就是，每一个消息可以有多个消费者，也就是说可以有多个接收方，如果希望发送的消息可以不被处理，或者被多个消息接收方处理就可以使用这种模式。类似于微信订阅号。

#### jms两种方式

##### 同步

订阅者或接收者调用receive方法来接收消息，receive方法在能够接收到消息之前（或超时之前）将一直阻塞 。（没什么应用价值）

##### 异步（应用较为广泛）

订阅者或接收者可以注册为一个消息监听器。当消息到达之后，系统自动调用监听器的onMessage方法。

### p2p模式实现

拉取jar包

```
<dependency>
      <groupId>org.apache.activemq</groupId>
      <artifactId>activemq-core</artifactId>
      <version>5.7.0</version>
    </dependency>
```

设置生产者

使用的工厂模式创建，代码基本是固定的。填入用户名和密码即可

```java
public class Producter {
    public static final String USERNAME="admin";
    public static final String PASSWORD="admin";
    public static final String BROKERAGE ="tcp://127.0.0.1:61616";
    public static final String QueueName ="myQueue";

    public static void main(String[] args) throws JMSException {
        start();
    }
    public static void start() throws JMSException {
        ActiveMQConnectionFactory activeMQConnectionFactory =
                new ActiveMQConnectionFactory(USERNAME, PASSWORD, BROKERAGE);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //消息的可靠性，自动签收
        Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);

        Queue queue = session.createQueue(QueueName);
        //创建一个生产者
        MessageProducer producer = session.createProducer(queue);
        //设置存放消息内容

        for (int i = 0; i < 5; i++) {
            TextMessage textMessage = session.createTextMessage("hello word this is one mq");
            System.out.println("消息队列存放内容");
            producer.send(textMessage);
            session.commit();
        }
         session.close();
        connection.close();


    }
}
```

设置消费者

```java
public class JmsReceiver {
    public static final String USERNAME="admin";
    public static final String PASSWORD="admin";
    public static final String BROKERAGE ="tcp://127.0.0.1:61616";
    public static final String QueueName ="myQueue";

    public static void main(String[] args) throws JMSException {
        start();
    }
    static public void start() throws JMSException {
        ActiveMQConnectionFactory activeMQConnectionFactory =
                new ActiveMQConnectionFactory(USERNAME, PASSWORD, BROKERAGE);
        Connection connection = activeMQConnectionFactory.createConnection();
        connection.start();
        //消息的可靠性，自动签收
        //Session session = connection.createSession(true, Session.AUTO_ACKNOWLEDGE);
        Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);

        Queue queue = session.createQueue(QueueName);
        MessageConsumer consumer = session.createConsumer(queue);
        //等待消息
        TextMessage receive = null;
        while (true) {
            receive = (TextMessage) consumer.receive();
            if (receive==null){
                break;
            }else {
                System.out.println("我是消费者收到的消息为"+receive.getText());
               // session.commit();
                //手动应答
                receive.acknowledge();
            }
        }
        session.close();
        connection.close();
    }
}


```

#### 创建session时是否创建事务

关闭事务，设置为false

```
 Session session = connection.createSession(false, Session.CLIENT_ACKNOWLEDGE);
```

开启事务，设置为true

```
 Session session = connection.createSession(true, Session.CLIENT_ACKNOWLEDGE);
```

当开启事务时必须手动提交，即session.commit();

#### 三种签收方式



1. **Session.AUTO_ACKNOWLEDGE** 

   当客户端从receiver或onMessage成功返回时，Session自动签收客户端的这条消息的收条。

   

 2.**Session.CLIENT_ACKNOWLEDGE**

客户端通过调用消息(Message)的acknowledge方法签收消息。在这种情况下，签收发生在Session层面：签收一个已经消费的消息会自动地签收这个Session所有已消费的收条。也就是必须手动应答 上述代码中receive.acknowledge();

 

3.**Session.DUPS_OK_ACKNOWLEDGE** 

Session不必确保对传送消息的签收，这个模式可能会引起消息的重复，但是降低了Session的开销，所以只有客户端能容忍重复的消息，才可使用。


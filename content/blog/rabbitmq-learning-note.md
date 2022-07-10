---
title: "RabbitMQ 基本实践"
date: 2022-07-09T16:43:26+08:00
draft: false
tags:
- rabbitmq
---

公司有一个项目要使用到 RabbitMQ, 本文是我自己学习 RabbitMQ 的记录. 

## 介绍一下 RabbitMQ
Erlang 语言实现 AMQP (Advanced Message Queuing Protocal) 的消息中间件

消息中间件的作用
 - 解耦: 可以不需要依赖下游的可用性, 提高单独项目的可用性. 
 - 冗余存储: 保存失败的消息
 - 拓展性
 - 削峰
 - 缓冲

### 结构

涉及的名词简单解释
- Producer
- Consumer
- Broker: 服务节点
- Queue: 内存存储消息数据的对象
- Exchange: 选择器, 生产者投递消息后, 提交给交换器, 由交换器根据`routingkey` 和 `bindingkey` 决定投递到哪个队列
	- RoutingKey: 生产消息提供`routingkey` 给交换器用于指定要投递的队列
	- BindingKey: 交换器, 通过 bindingkey 与对应的队列关联起来
- Connection: 客户端与 Broke 建立的 TCP 连接
- Channel: 是建立在 Connection 上抽象的虚拟连接. 实现多线程可以 TCP 连接, 多个信道可能复用同一个 TCP 连接


交换器的类型

- fanout : 投递到所有队列
- direct: 投递到 `routingkey`  和 `bindingkey` 匹配的
- topic: direct 的拓展, 支持模糊匹配
- headers : 性能差, 少用

消息投递流程:

1. 生产者连接到 Broker, 开启信道
2. 生产者声明一个交换器
3. 生产者声明一个队列
4. 生产者通过路由键将交换器和队列绑定
5. 生产者发送消息到 Broker 
6. 交换器根据接受对路由键匹配队列
7. 投递到对应的消息队列
8. 如果没有匹配的队列, 丢弃或者退回给生产者


消费者接受消息流程:

1. 建立连接, 开启信道
2. 消费者向 Broker 发起消息请求
3. Broker 回应并返回消息
4. 消费者发送确认 (ack) 接受到消息
5. 关闭信道
6. 关闭连接

### AMQP

位于应用层的通信协议 ( 在 TCP 之上, 将数据填充到 TCP 中)

几个基础的协议定义的操作
- Protocal Header 0-9-1 指定协议
- Connection.Start
- Channel.Open
- Basic.Publish 推送消息
- Channel.Close
- Connection.Close


## 部署
在你的机子上部署一个玩具吧

采用 docker 部署
`docker run -itd --name rabbitmq -p 5671:5671 -p 5672:5672 -p 4369:4369 -p 25672:25672 -p 15671:15671 -p 15672:15672 rabbitmq:management`

进入管理页面
访问 http://127.0.0.1:15672/

登入
用户名: guest
密码: guest 

[更多默认配置](https://www.rabbitmq.com/configure.html#supported-environment-variables)


## 管理

进入交互 shell
`docker exec -it rabbitmq bash`

增加一个用户
`rabbitmqctl add_user ian ian1234`

增加一个 vhost
`rabbitmqctl add_vhost playground`

vhost是什么? 
vhost (virtual host), 虚拟主机, 在实例间提供逻辑上的分离 -- 实现数据隔离.
RabbitMQ 默认创建一个名为 "/" 的 vhost

查看 vhost
`rabbitmqctl list_vhosts`

设置权限
`rabbitmqctl set_permissions --vhost playground ian ".*" ".*" ".*"`

**amqp uri规则**
`"amqp://userName:password@ipAddress:portNumber/virtualHost"`
根据我们上述的新增在用户和vhost, 可以得到uri:
`amqp://ian:ian1234@localhost:5672/playground`
5672为 rabbitmq的默认终端, 我们的 docker contain 需要把它映射到本机带端口

## 实现生产者

go 使用 `github.com/rabbitmq/amqp091-go` 包

根据上文的消息投递流程实现

连接到 broker
``` go
	connectionUrl := "amqp://ian:ian1234@localhost:5672/playground"
	conn, err := amqp.Dial(connectionUrl)
``` 

打开 channel
```go
	ch, err := conn.Channel()
```

声明一个交互器( 该步骤可以省略, 直接使用默认的 `direct`交换器)
``` go
	err = ch.ExchangeDeclare("hello-exchange", "direct", true, false, false, false, nil)

```

声明一个队列
``` go
	q, err := ch.QueueDeclare("hello", true, false, false, false, nil)
```

将 Exchange 绑定到队列上 (如果声明交换器的步骤省略了, 该步骤也可以省略)
``` go
	err = ch.QueueBind(q.Name, "hellokey", "hello-exchange", false, nil)
```

发送消息 (如果没有声明交换器, 对应的参数直接传入空字符串`""`, 会使用默认的 `direct` 交换器)
``` go
		err = ch.Publish("hello-exchange", "hellokey", false, false, amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
```

[完整代码地址](https://github.com/ynikl/rabbitmq-demo/blob/main/cmd/producer/main.go)

登录到本地管理页面可以查看类似于下图, 有消息投递

![生产消息成功](/rabbitmq-producer-manager-pic-20220704.png)


## 实现消费者

消费者相对于生产者就简单多了.
打开信道直接消费就可以了. 

连接, 打开信道
``` go
	connectionUrl := "amqp://ian:ian1234@localhost:5672/playground"
	conn, err := amqp.Dial(connectionUrl)
	if err != nil {
		log.Fatal(err)
	}
	defer conn.Close()

	ch, err := conn.Channel()
```

开始消费
``` go
	// msgsCh 是一个消息管道
	msgsCh, err := ch.Consume("hello", "consumer-name", true, false, false, false, nil)
	for msg := range msgsCh {
		log.Println("received a message: ", string(msg.Body))
	}

	log.Println("done, msg channel is closed")
```

![消费成功](/rabbitmq-receive-success-20220709.png)


## 参考

- [官网 tutorial](https://www.rabbitmq.com/tutorials/tutorial-one-go.html)
- [RabbitMQ 实战指南](https://book.douban.com/subject/27591386/)




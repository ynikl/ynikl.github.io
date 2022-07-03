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
`rabbitmqctl add_user ian 123456`

增加一个 vhost
`rabbitmqctl add_vhost playground`

vhost是什么? 
vhost (virtual host), 虚拟主机, 在实例间提供逻辑上的分离 -- 实现数据隔离.
RabbitMQ 默认创建一个名为 "/" 的 vhost

查看 vhost
`rabbitmqctl list_vhosts`

设置权限
`rabbitmqctl set_permissions --vhost playground ian ".*" ".*" ".*"`



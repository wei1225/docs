#rabbitmq基础知识

by can.  
2012/9/6

##简介
[rabbitmq](http://www.rabbitmq.com)是[AMQP](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol)的一个开源实现,openstack就是使用rabbitmq作为基础的.

AMQP是一个所谓"线路协议",即规定了消息怎样在网络中传递的协议.利用rabbitmq可以很方便地在不同的节点/进程之间传递消息

##重要术语
* queue: 消息队列
* exchange: 基于一定规则把消息分发到不同的queue中,可以看作是对消息的"路由器".有以下4种:
	* fanout: 向它所连接的所有queue广播消息
	* direct: 根据routing_key精确转发消息
	* topic: 根据routing_key模糊转发消息,支持"*" "#"通配符
	* headers: 根据header转发信息
* bind: exchange用以分发消息到queue中的规则

更详细的基本概念可以参考[AMQP concepts](http://www.rabbitmq.com/tutorials/amqp-concepts.html)

##工作流程
* 发端(publisher):
	1. 连接到rabbitmq服务器
	2. 声明相应的exchange/queue/bind,或使用已有/预定义的
	3. 发送消息
	4. 等待ACK(如果需要的话)
* 收端(consumer):
	1. 连接到rabbitmq服务器
	2. 声明相应的exchange/queue/bind,或使用已有/预定义的
	3. 声明回调函数(所等待消息到达时触发的函数)
	4. 监听等待消息
	
##代码示例

参见[rabbitmq tutorial](http://www.rabbitmq.com/getstarted.html)

##应用场景

* 传递消息(废话)
* 任务分发(类似生产者-消费者模型)
* 消息的按内容路由/分发
* RPC(远程过程调用)
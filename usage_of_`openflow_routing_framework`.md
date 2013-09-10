#Usage of `OpenFlow Routing Framework`
by can.  
2013/9/10

## 获取代码

在某一文件夹下执行`git clone https://github.com/cannium/openflow_routing_framework.git`, Ubuntu应该自带git。因为我仍然在不停的修改,以后如果要更新,在文件夹下执行`git pull`

安装依赖，想到的主要有`eventlet`,`greenlet`,`ipdb`,`ryu`,一般都可以用`pip install xxx`安装

## 配置执行

目前有两个配置文件，`routing.config`和`bgper.config`

`routing.config`用来配置网络拓扑中openflow switch的某些有IP地址的端口的IP地址，以及是否作为整个OpenFlow网络的边界路由器

`bgper.confg`配置和BGP相关的信息

运行使用命令`sudo ryu-manager --observe-links ryu.topology.switches routing.py bgper.py`，然后openflow switch就可以连接到该控制器了

## 代码概览

`BGP4.py`
BGP包的解析与封包

`algorithm.py`
路由算法，重点关注`find_route`函数，接受`src`/`dst`,返回一个switch object的列表

`bgp_server.py`
BGP的协议控制，建立连接，收发报文等

`bgper.py`
与ryu交互，初始化bgp_server，回答`destination_request`，即询问OpenFlow网络以外的目标的路由

`convert.py`
各种地址转换

`dest_event.py`
`bgper.py`里所用的Event类

`gateway.py`
类定义

`route_entry.py`
类定义

`routing.py`
OpenFlow网络内部路由，主要处理发送到控制器的包，并且下发流表。可重点关注函数`deploy_flow_entry`

`switch.py`
switch和port类定义

`tap.py`
读写tap接口

`utils.py`
解析配置文件

## 参考

当时的[设计文档](https://github.com/cannium/docs/blob/master/the_openflow_routing_project.md)
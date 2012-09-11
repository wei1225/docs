#openstack开发环境

基于IBM的__*OpenStack - Community Contributions v2*__文档

by can.  
2012/9/5

##操作系统
本次安装采用Ubuntu 12.04,理论上11.10也会相当顺利

##devstack
[devstack](http://devstack.org)是用于快速部署openstack开发环境的一组脚本

安装前先阅读以下官方文档:

1.[All-In-One: Dedicated Hardware](http://devstack.org/guides/single-machine.html):介绍单机安装的文档

2.[localrc](http://devstack.org/localrc.html):介绍配置文件localrc的文档

我的配置文件localrc如下("#"后面的内容是注释):

```
FLOATING_RANGE=210.25.137.224/27
FIXED_RANGE=10.0.0.0/24 
FIXED_NETWORK_SIZE=256
FLAT_INTERFACE=eth0
ADMIN_PASSWORD=openstack
MYSQL_PASSWORD=buptnic
RABBIT_PASSWORD=openstack
SERVICE_PASSWORD=$ADMIN_PASSWORD
HOST_IP=210.25.137.233

# openstack install directory:
DEST=/opt/stack
# devstack log file:
LOGFILE=stack.sh.log
# screen log file:
SCREEN_LOGDIR=$DEST/logs/screen
# syslog
SYSLOG=True
SYSLOG_HOST=$HOST_IP
SYSLOG_PORT=516
# reclone openstack every time
RECLONE=yes
# enable swift
ENABLED_SERVICES+=,swift
SWIFT_HASH=66a3d6b56c1f479c8b4e70ab5c2000f5
SWIFT_REPLICAS=1
SWIFT_DATA_DIR=$DEST/data
# SQL service catalog
KEYSTONE_CATALOG_BACKEND=sql
# API rate limit
API_RATE_LIMIT=False
# enable quantum
disable_service n-net
enable_service q-svc
enable_service q-agt
enable_service q-dhcp
enable_service quantum
LIBVIRT_FIREWALL_DRIVER=nova.virt.firewall.NoopFirewallDriver
Q_PLUGIN=openvswitch
NOVA_USE_QUANTUM_API=v2
Q_AUTH_STRATEGY=keystone
SERVICE_TOKEN=openstack

```
注意quantum部分以后可能会有大的变化

写好配置文件之后执行`stack.sh`即可启动openstack,执行`unstack.sh`则关闭之

##eclipse
安装只需`sudo apt-get install eclipse`,然后等待

装完之后打开help > install software,安装pyDev和Egit.然后导入openstack项目

推荐以下相关文档:

1.[pyDev getting started](http://pydev.org/manual_101_root.html)

2.[Setup keystone in eclipse](http://wiki.openstack.org/Setup%20keystone%20in%20Eclipse):这篇是说怎么导入keystone的,其它项目类似

##FAQ
* 怎么远程登陆到服务器打开eclipse?

使用ssh时添加`-X`参数:`ssh user@aaa.bbb.ccc.ddd -X`,就可以在本机打开服务器上的窗口了.
用Windows的同学请先自行配置ssh和X window

* `stack.sh`运行失败了怎么办?

我很想简单的讲如何如何即可,但是…

鉴于openstack是一个正在不断发展的项目/鉴于服务器的网段经常抽风/鉴于每个人的人品不一样,有以下建议:  
		* 	尽量使用全新安装的系统  
		*	系统语言使用英语,这样错误信息容易搜索到  
		*	在学校使用[https://ipv6.google.com/ncr](https://ipv6.google.com/ncr)作为默认搜索引擎  
		*	多扶老奶奶过马路攒人品












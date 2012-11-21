# Open vSwitch安装
2012.11.21 by can.

## 安装环境
Ubuntu 12.04 LTS (server edition)

## apt-get包安装
参考
[[1]](http://networkstatic.net/openvswitch-configure-from-packages-and-attaching-to-a-floodlight-openflow-controller/)
[[2]](http://blog.scottlowe.org/2012/08/17/installing-kvm-and-open-vswitch-on-ubuntu/)

目前支持ovs 1.4版本

## 手工包安装

以下操作大多需要root权限

### 下载

去[官网](http://openvswitch.org/download/)寻找最新版本并下载,本文版本为1.7.1

### 安装依赖
```
apt-get install python-simplejson python-qt4 python-twisted-conch automake autoconf gcc uml-utilities libtool build-essential autoconf automake pkg-config libssl-dev iproute tcpdump module-assistant

apt-get install linux-headers-`uname -r`

```
### 解压缩,改名,编译deb包

```
tar xvf openvswitch-1.7.1.tar.gz

mv openvswitch-1.7.1.tar.gz openvswitch_1.7.1.orig.tar.gz

cd openvswitch-1.7.1

dpkg-buildpackage
```
注意文件名里的版本号

### 安装

这时在`openvswitch-1.7.1`文件夹外会出现编译好的deb包,安装之(`dpkg -i 包名称`)

在安装了`openvswitch-datapath-source_1.7.1-1_all.deb`前提下,编译安装内核模块:

```
module-assistant auto-install openvswitch-datapath
```

### 启动
```
service openvswitch-switch start
```

也可以开启ovs与linux bridge的兼容模式,修改`/etc/default/openvswitch-switch`,把`#BRCOMPAT=no`改成`BRCOMPAT=yes`
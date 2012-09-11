#系统监控与图形化

使用collectd

by can.  
2012/9/5

##简介
[collectd](http://collectd.org)是一个监控系统状态的daemon,它周期性地收集系统参数,比如CPU/内存/网络使用情况,生成rrd文件.

rrd文件可以使用[RRDtool](http://oss.oetiker.ch/rrdtool/)这个库方便地读取,因此collectd有很多的图形前端,包括本地应用程序和网页程序

##collectd
安装:

```
sudo apt-get install collectd
```

配置文件位于`/etc/collectd`;
监控信息(rrd文件)位于`/var/lib/collectd/rrd`

collectd可以将一台或多态主机配成服务器从而接收其它主机的状态信息,以实现对服务器群的监控,详细可以参考[Networking introduction](http://collectd.org/wiki/index.php/Networking_introduction)

##图形前端

这个[页面](http://collectd.org/wiki/index.php/List_of_front-ends)列出了所有可用的前端,我没有逐一尝试.亲测Collectd Graph Panel(CGP)和kcollectd可用

###kcollectd
这是一个本地应用程序,使用Qt编写

安装:

```
sudo apt-get install kcollectd
```
###Collectd Graph Panel(CGP)

这是一个网页程序,需要先安装一个网页服务器(nginx和apache皆可),然后配置安装CGP.具体操作可以参考[collectd详解](http://www.drupal001.com/2012/07/system-monitor-collectd/)

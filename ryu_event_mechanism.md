#Ryu event mechanism
by can.  
2013/3/12


## 版本及相关信息

基于Ryu 1.7(git commit `914826d 474285f0b1b040020385b10312de5b69f`)

github地址:[https://github.com/osrg/ryu](https://github.com/osrg/ryu) 下文约定该文件夹为`/`

## 相关第三方库
### gevent
*关键字:*  

1. 协程(coroutine)  
2. 并发

### oslo.config
*关键字:*

1. 来自OpenStack项目
2. 处理参数、配置等


## Walk through
ryu提供的启动命令是:`ryu-manager /path/to/app1.py app2.py …`,故程序的入口位于`/bin/ryu-manager`文件,以下是对该文件的分析

```
app_lists = CONF.app_lists + CONF.app +['ryu.controller.ofp_handler']
```  

把`ryu.controller.ofp_handler`模块加入要启动的app列表；  
`CONF.app_lists`和`CONF.app`是从用户命令行输入获取的模块名称

```
    app_mgr = AppManager()
    app_mgr.load_apps(app_lists)
    contexts = app_mgr.create_contexts()
    app_mgr.instantiate_apps(**contexts)
```
初始化一个`AppManager`,代码位于`/ryu/base/app_manager.py`;  
载入并且实例化`app_list`中的`app` (`RyuApp`的子类),以及位于`app._CONTEXTS`(它的另一个名字是 `app.context_iteritems()`)中的模块.

`AppManager.load_apps`/`AppManager.create_contexts`等模块下文有详述

```
    services = []

    ctlr = controller.OpenFlowController()
    thr = gevent.spawn_later(0, ctlr)
    services.append(thr)
```

实例化并且执行了`OpenFlowController`(代码位于`/ryu/controller/controller.py`),并且添加到`services`列表里.

```
    webapp = wsgi.start_service(app_mgr)
    if webapp:
        thr = gevent.spawn_later(0, webapp)
        services.append(thr)
```
`wsgi.start_service()`在`app_mgr.contexts.values()`中寻找`WSGIApplication`并且使用`WSGIServer(instance)`启动之

```
    gevent.joinall(services)
    app_mgr.close()
```
等待所有`services`执行结束,然后执行`close()`进行清理

## 部分模块详述
请对照代码查看
###ryu.base.app_manager.AppManager:
####load_app
根据模块名称获取模块,并且从模块中找出所有`RyuApp`的子类,存入`clses`,并且返回字典序最小的模块(这意味着写模块的时候尽量一个`.py`里只包含一个`RyuApp`的子类)

[inspect.getmembers() 文档](http://docs.python.org/2/library/inspect.html#inspect.getmembers)
####load_apps
从`app_list`中获取app名称,调用`load_app`获取其中的RyuApps,存入`application_cls`字典,同时把RyuApp的context存入`contexts_cls`字典
####create_contexts
从`contexts_cls`中取出各个类,并且实例化,然后加入`contexts`这个字典.最后返回`contexts`
####instantiate_apps
此函数接受`contexts`字典;  
从`applications_cls`中取出各个类,用`contexts`作为参数实例化,注册该app(`register_app`见下文),并且把app加入`applications`字典;  
从`SERVICE_BRICKS`中取出各个app,根据app中函数的参数(`observer`和`ev_cls`)确定出事件和类,调用`register_observer`:从这里可以看出,ryu可以自动处理两种事件传递.1)`handler.observer`在`SERVICE_BRICKS`里 2)`handler.ev_cls`在`brick._EVENTS`里

###ryu.base.app_manager:
####register_app
把app加入`SERVICE_BRICKS`字典,并注册相应实例(`register_instance`见下文)

###ryu.controller.handler:
####register_instance
此函数接受一个RyuApp实例;  
枚举实例中的函数,如果函数有参数`ev_cls`(表示是一个handler),则进行`register_handler`

###ryu.base.app_manager.RyuApp:
####register_handler
接受两个参数,事件类和handler函数;  
把handler附加到`event_handlers`字典相应事件的列表中.
`event_handlers`的结构是: `event_handler[event class] = [handler 1, handler 2, …, handler n]`
####register_observer
接受3个参数,事件类(`ev_cls`),观察该事件的类的名字(`name`),状态(dispatcher).使用`observers`字典,结构是:`observers[ev_cls][name] = dispatcher`,举个栗子:`{<class 'ryu.controller.ofp_event.EventOFPPortStatus'>: {'SimpleSwitch': ['main']}, <class 'ryu.controller.ofp_event.EventOFPPacketIn'>: {'SimpleSwitch': ['main']}}`

##事件机制
###当`@set_ev_cls`时发生了什么
这是一个修饰函数,定义在`/ryu/controller/controller.py`  

关于python中的decorator,[参见这里](http://stackoverflow.com/questions/739654/understanding-python-decorators)  

简单的说,`@set_ev_cls`这个修饰器给handler函数添加了三个参数:`ev_cls`,`dispatchers`,`observer`.用于上文的`register_handler`和`register_observer`
###事件的发生
有两种方式:

1. `RyuApp.send_event_to_observers(event, state)`
2. OpenFlow的事件,如`ofp_event.EventOFPPacketIn`,是`/ryu/controller/controller.py`的`_recv_loop`函数中,由switch产生的数据包触发的

###举个栗子
```
import gevent

from ryu.base import app_manager
from ryu.controller import event
from ryu.controller.handler import set_ev_cls


TEST_EVENT_EV_DISPATCHER = "test_event"


class TestEvent(event.EventBase):
    def __init__(self, msg):
        super(TestEvent, self).__init__()
        self.msg = msg


class Test(app_manager.RyuApp):	
    _EVENTS = [TestEvent]
    def __init__(self):
        super(Test, self).__init__()
        gevent.spawn_later(0, self._send_event_loop)

    def _send_event_loop(self):
        i = 0
        while True:
            self.send_event_to_observers(TestEvent(i))
            i += 1
            gevent.sleep(1)

    @set_ev_cls(TestEvent, TEST_EVENT_EV_DISPATCHER)
    def _recv_handler(self, ev):
        print 'recv:', ev.msg
```

注意类`Test`的类属性`_EVENTS`表示这个类将会产生的事件列表,定义这个列表后ryu会自动关联消息的发生者和观察者,关键代码在(上文)`ryu.base.app_manager.AppManager.instantiate_apps`

如果不定义`_EVENTS`,则可以直接在`Test`的`__init__`里调用`self.register_observer(TestEvent, self.name)`手工关联








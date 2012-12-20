# OSPF basics
by can.  
2012/12/19

## 版本说明
主要基于OSPFv2(RFC 2328),参考了OSPFv3(IPv6 for OSPF, RFC 5340)

## 基本概念

### stub network / transit network  
transit network: a network having two or more attached routers

### autonomous system(AS)  
自治系统,网络管理单位

### area 区域  
在OSPF里,把AS划分成area以减少路由信息造成的网络流量.area包括router和与之相连的network 

area有以下分类:  

*  stub area  
不接收AS外路由信息通告,路由是基于默认路由(default route)
*  backbone area  
	- area号保留为0
	- 所有其它area都与backbone相连
	- 负责分发路由信息
	- 所有部分必须直接相连,但不必物理上相连(可配置virtual link)
* transit area  
允许流量穿过(允许源/目的地址不在area的包通过)
* NSSA(not-so-stub-area)  
(?) 通过定义一个只能在NSSA内扩散的type 7 LSA,进行有限的AS外路由扩散.[RFC 3101]

### OSPF网络分类
* broadcast  
支持广播的网络,例如以太网(ethernet)
* point-to-point(p2p)  
点到点网络,端口一般不需配置地址
* point-to-multipoint(p2mp)  
点到多点,可以看作一组p2p的链路
* NMBA(non-broadcast multiple access)  
多址接入但没有广播机制的网络

### neighbor / adjacency
有连接在同一子网上的接口(interface)两个路由器称为neighbor. adjacency比neighbor更紧密,二者可以进行路由信息交换

### 路由器分类
按照区域划分

* ABR: area border router,可能会是多个area的成员
* ASBR: AS border router
* IR: internal router
* BR: backbone router

按照在子网中的职能

* DR: designated router,指派路由器
* BDR: backup DR

DR/BDR实际是路由器的某个接口(interface),只存在于broadcast/NMBA网络上

选择DR之后,子网中的其它路由器将与之建立adjacency关系

### OSPF packet type
* Hello, 用于邻居发现,建立邻接关系
* DD(database description)
* LSR(link state request)
* LSU(link state update,包含LSA)
* LSAck(link state acknowledgement)

### link state advertisement(LSA)
OSPFv2:

* LSA 1: router-LSA  
	- (产生者)所有路由器
	- (通告内容)area内路由器各个接口状况
	- (广播区域)路由器所在area
* LSA 2: network-LSA
	- 子网的DR
	- 子网上连接的路由器
	- 子网所在area
* LSA 3: summary-LSA
	- ABR
	- 到达某网络A的路由
	- 除A所在area外,ABR所在的area
* LSA 4: summary-LSA
	- ABR
	- 到达某ASBR的路由
	- 除ASBR所在area外,ABR所在的area
* LSA 5: AS-external-LSA
	- ASBR
	- 到达AS外地址的路由
	- 整个AS

OSPFv3:

* LSA 1: router-LSA
* LSA 2: network-LSA
* LSA 3: inter-area-prefix-LSA
* LSA 4: inter-area-router-LSA
* LSA 5: AS-external-LSA
* LSA 8: link-LSA
* LSA 9: intra-area-prefix-LSA

其中,LSA 1-5与OSPFv2相同

### AllSPFRouters / AllDRouters
两个多播地址,分别代表所有SPF路由器和所有DR  
IPv4地址分别是`224.0.0.5`和`224.0.0.6`  
IPv6地址分别是`ff02::5`和`ff02::6`

## 拓扑示例

RFC 2328, figure 6

```


             ...........................
             .   +                     .
             .   | 3+---+              .      N12      N14
             . N1|--|RT1|\ 1           .        \ N13 /
             .   |  +---+ \            .        8\ |8/8
             .   +         \ ____      .          \|/
             .              /    \   1+---+8    8+---+6
             .             *  N3  *---|RT4|------|RT5|--------+
             .              \____/    +---+      +---+        |
             .    +         /      \   .           |7         |
             .    | 3+---+ /        \  .           |          |
             .  N2|--|RT2|/1        1\ .           |6         |
             .    |  +---+            +---+8    6+---+        |
             .    +                   |RT3|------|RT6|        |
             .                        +---+      +---+        |
             .                      2/ .         Ia|7         |
             .                      /  .           |          |
             .             +---------+ .           |          |
             .Area 1           N4      .           |          |
             ...........................           |          |
          ..........................               |          |
          .            N11         .               |          |
          .        +---------+     .               |          |
          .             |          .               |          |    N12
          .             |3         .             Ib|5         |6 2/
          .           +---+        .             +----+     +---+/
          .           |RT9|        .    .........|RT10|.....|RT7|---N15.
          .           +---+        .    .        +----+     +---+ 9    .
          .             |1         .    .    +  /3    1\      |1       .
          .            _|__        .    .    | /        \   __|_       .
          .           /    \      1+----+2   |/          \ /    \      .
          .          *  N9  *------|RT11|----|            *  N6  *     .
          .           \____/       +----+    |             \____/      .
          .             |          .    .    |                |        .
          .             |1         .    .    +                |1       .
          .  +--+   10+----+       .    .   N8              +---+      .
          .  |H1|-----|RT12|       .    .                   |RT8|      .
          .  +--+SLIP +----+       .    .                   +---+      .
          .             |2         .    .                     |4       .
          .             |          .    .                     |        .
          .        +---------+     .    .                 +--------+   .
          .            N10         .    .                     N7       .
          .                        .    .Area 2                        .
          .Area 3                  .    ................................
          ..........................

```
其中,  
- AS被划分为4个area(area0 - area3)  
- 类似N1/N2形状的网络是stub network  
- 类似N3形状的网络是transit network  
- 类似RT3/RT4的路由器是ABR  
- RT5和RT7是ASBR  
- RT3/4/5/6/7/10/11是BR(RT11通过virtual link连接到RT10,所有ABR都是BR)  

## 路由信息交换

路由器启动之后,首先使用Hello packet探测与之相连的路由器(neighbor),在broadcast和NBMA网络中,Hello protocol还兼具选举DR/BDR的功能.

选举出DR/BDR之后,DR/BDR与其它路由器建立adjacency关系,只有建立adjacency关系的路由器之间才真正进行路由信息同步;在p2p/p2mp/virtual link中,任何直接通信的neighbor都成为adjacency.

当两个neighbor路由器尝试建立adjacency关系时,它们首先通过DD packet向对方描述自己的路由数据库(只发送LSA header).如果发现了对方的数据库较新,就将之放入LS request队列中.之后双方通过LSR-LSU进行LSA的同步.同步完成后,这个adjacency被视为完整建立.

之后若存在路由更新,通过LSU完成

在broadcast network中,路由器会利用multicast能力进行路由信息扩散

## 路由计算
路由器根据收集到的link state信息,建立一棵以自己为根的最短路径树,亦即路由表

## 细节(i.e.需要详读RFC的内容)
- 路由器interface的状态及转换
- neighbor的状态及转换
- 各个type的LSA的内容及报文格式

#ToMaTo安装

by can.

2012/9/11

##简介

ToMaTo 是"Topology Management Tools"的缩写,其[官方主页](http://tomato.german-lab.de),[github主页](https://github.com/dswd/ToMaTo/wiki)

本文主要介绍其前端(tomato-web)和后端(tomato-backend)的安装.

##解决依赖关系

首先应安装该项目的依赖.如下:

注: `python-django`不要用`apt-get`安装,兼容性有问题,我测试了手工安装1.3版本可用

* tomato-web依赖:

```
python, python-django (>= 1.2), python-simplejson, apache2, libapache2-mod-python
```

* tomato-backend依赖:

```
debconf (>= 0.5) | debconf-2.0, python, python-twisted-runner, python-django (>= 1.2), python-django-south (>= 0.7), python-ldap, python-twisted-web, openssh-client, adduser, daemon, rsync, python-psycopg2, postgresql, dbconfig-common
```

使用`apt-get install xxx`即可


##通过repository安装

详见[ToMaTo installation](https://github.com/dswd/ToMaTo/wiki/Tomatoinstallation)

注意第二步会报错,这个错由第三步解决…

##通过deb包安装(不推荐)

首先在[deb包下载页面](https://github.com/dswd/ToMaTo/downloads)下载相应的安装包.

使用`dpkg -i xxx.deb`即可安装

##还需解决一个bug...
issue[链接](https://github.com/dswd/ToMaTo/issues/160)

如果按照原来步骤,认证部分是有问题的.这是因为`htpasswd`工具改变了密码生成的算法.如果要生成与ToMaTo兼容的密码,可以用python的交互式命令行这样生成:

```
Python 2.7.1 (r271:86832, Jun 16 2011, 16:59:05) 
[GCC 4.2.1 (Based on Apple Inc. build 5658) (LLVM build 2335.15.00)] on darwin
Type "help", "copyright", "credits" or "license" for more information.

>>> import crypt
>>> crypt.crypt('buptnic','v4')
'v4kBcARz3XYr2'
>>> crypt.crypt('buptnic','hh')
'hh.31mj7rlXX.'

```
`crypt()`接受两个参数,前一个是密码的明文,后一个是一个两位的[salt](http://en.wikipedia.org/wiki/Salt_(cryptography\)),可以任意设定

得到加密的密码之后,将其写入`/etc/tomato/users`文件,格式是`用户名:加密的密码`


例如这个设定就可以用admin/buptnic登陆:

```
admin:v4kBcARz3XYr2
```









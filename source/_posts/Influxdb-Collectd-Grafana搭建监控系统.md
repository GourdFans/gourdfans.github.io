---
title: Influxdb+Collectd+Grafana搭建监控系统
date: 2017-12-14 00:01:29
tags: 监控
---

上次开发了一个秒杀类似的业务，活动开始以后对系统监控，除了用阿里的Tsar来查看系统之外，还需要监控数据库，Redis之类的等等。一个窗口根本看不过来，所以就想着有没有监控系统来能够一眼看到系统的具体情况，在公司有一套监控系统，经过审查代码发现用的是Grafana来搭建的。经过调研发现线程的系统有两套 ELK或者是Grafana + Influxdb + Collectd来搭建。思路都差不多，每台机器上安装上收集器，把收集到的信息存放到ES或者时序数据库里面，然后通过Grafana或者Kiban来做前端展示。感觉Grafana比较简单一点，于是使用这套系统来搭建。

首先是Grafana安装，可以参考官方文档进行安装都可以了，不过是国内下载比较慢，需要点时间才能下载下来。Centos安装

```
sudo yum install https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.6.2-1.x86_64.rpm
```

安装完毕把应用加入到启动列表里面

```
sudo /bin/systemctl daemon-reload
sudo /bin/systemctl enable grafana-server.service
```

然后就可以启动Grafana了

```
sudo /bin/systemctl start grafana-server.service
```

启动以后的Grafana会在3000端口，访问3000端口就可以访问系统了，接下来就是配置时序数据库,安装Influxdb，然后配置图标即可。

Influxdb安装也可以参考官方文档，是一个Go语言来编写的时序数据库，关于时序数据库的概念，可以去Google一下，其实最主要的一点就是时间递增的，每条记录都有时间，这样能够清楚的知道一个数据的变化过程。Influxdb现在版本在1.。不过我安装的是1.0版本的，安装也特别简单，参考官网即可：

```
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.0.0.x86_64.rpm
sudo yum localinstall influxdb-1.0.0.x86_64.rpm
```

在启动之前需要修改一下Influxdb配置，influxdb.conf。因为默认Collectd是关闭的，设置为开启即可，如果是内网绑定一下内网地址

```
[[collectd]]
  enabled = true
  bind-address = "127.0.0.1:25826"
  database = "collectd"
 
```

配置完毕以后启动influxdb:

```
service influxdb start
```

influxdb会在8083端口打开一个web的操作界面，可以通过web页面去操作数据，插入或者查询数据等操作，如果能够打开，说明也已经启动成功了。

关于Collectd的安装，没有其他的参考官网安装即可，只需要在配置的时候，引入Network的Plugin并且配置Network地址：

```
<Plugin network>
        Server "127.0.0.1" "25826"
</Plugin>
```

25826是influxdb监听的端口，Collectd会把指定的数据写入到该端口，Influxdb会监听并把数据写入到数据库中.

上面的都安装完毕以后，就需要在Grafana中配置DB，配置相应的图表。对于图表的配置需要花点时间去摸索，里面有很多函数，而且数据收集的单位之类的都不尽相同，现在之只能够搭建简单的监控，Collectd有很多插件可以去监控Mysql，redis等等。如果不满足要求，可以通过Influxdb的Client去收集自己系统中运行的数据，比如QPS、RT等信息。都可以用该系统来实现了。

在搭建系统的过程中，有个地方是在配置时候花费比较多的时间的。第一个是Influxdb数据库到底怎么用，因为不同于Mysql中db和table的概念，在Influxdb中是metric和tag以及key和value的概念。所以如果想要使用该系统，需要对Influxdb有个大致的了解，可以先不用去看具体是如何实现的，数据是如何存储的等等信息，但是基本的概念约定要定清楚，不然像我一样连数据都写入不进去，就算写入进去了，不会查找和查看，走了很多的弯路。

另外一个花费比较多时间的是，在Grafana创建图表的过程中，如何把值以你自己需要的方式展现出来，这也是一个不小的挑战，第一点需要搞清楚你到底想要看到那些数据，比如我想看到内存的使用量。那么第步就是到Influxdb中查询出需要的数据，查询是一条SQL语句，图表配置也是配置一个SQL语句的过程，所以如果能从数据库中正确的查询出自己需要的数据，图标按照查询语句配置就行了。数据查询出来以后会有单位的问题，还有就是是否直接显示、显示成波浪线还是什么之类的，xy各自标示什么意思等等，这些也都需要花点时间去摸索。自己如果想要收集数据去展示，最好先想好自己要收集那些数据，tag和key-value如何设计，图标怎么去展示想好以后再去动手。

关于Influxdb + Grafana + collectd的介绍先介绍到这里，网上其他地方也有很多的教程，在配置过程中可以多出参考，不过官网上都介绍的挺详细的，跟着官网去配置一般都不会有太大的问题。自己搭建的一张图 ![系统监控](Influxdb+Collectd+Grafana搭建监控系统/Grafana.png)


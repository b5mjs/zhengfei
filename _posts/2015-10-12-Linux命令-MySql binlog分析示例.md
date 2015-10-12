---
layout: post
title: Linux命令-MySql binlog分析示例
date: 2015-10-12 13:43:07
tags: [Linux命令]
published: True

---

话说公司项目用到阿里开源项目[Otter](https://github.com/alibaba/otter),一直运行没有问题,突然今天出现了数据不一致的情况,主库数据正常,从库数据出现了重复,问从库的业务方是否自己人工操作或者有程序在后边运行没,说他们那边没有的,所以想来想去,还是看binlog是否有操作过,如果binlog存在记录,那就说明同步出现了问题,如果不存在,肯定是业务方那边有数据操作,废话不多说,开始操作.

#Step-1:找到binlog日志目录
一般mysql的配置文件在/etc/my.cnf

```shell
cat /etc/my.cnf
```

![/etc/my.cof]({{site.baseurl}}/assets/img/2015-10-12/2015-10-12 14:24:44.png)
log_bin对应的参数就为binlog文件存放的目录(/data0/db-binlog/test/passport/)

#Step-2: 使用mysqlbinlog命令提取二进制日志
我的测试环境binlog日志并不多,定期的会删除,如下图
![binlog日志列表]({{site.baseurl}}/assets/img/2015-10-12/2015-10-12 14:38:49.png)

``` shell
mysqlbinlog --start-datetime='2015-04-22 20:51:00' --stop-datetime='2015-04-22 21:01:00' --base64-output -v mysql-bin.000022 > temp-file.1
```

\--start-datetime:sql执行开始时间  
\--stop-datetime:sql执行结束时间  
mysql-bin.000022:binlog日志文件名   
temp-file.1:转换后的日志文件名  

例如:将多个binlog二进制日志文件提取到zhengfei.tmp.1文件
![binlog日志列表]({{site.baseurl}}/assets/img/2015-10-12/2015-10-12 15:22:56.png)

注意:binlog日志很大的话,执行此命令比较慢,请耐心等待,如果有好的方法的话可以推荐给我,多谢啦!!!

#Step-3:从日志文件中过滤自己有用的信息

转换后的文件我们就可以通过head tail tailf等命令查看了,如果日志文件很大的化,不要用vi vim cat等命令查看,结果么,你懂的

我现在要查对某个数据库的某张表执行sql insert语句,如下:

```shell
grep -A 35 '### INSERT INTO `test_db_name`.`test_table_name`' zhengfei.tmp.1 > zhengfei.tmp.2
```

\-A:匹配到这行数据向下35行都显示(数字根据自己的数据结构自己调整)  
单引号里面内容为匹配内容  
zhengfei.tmp.1:要匹配的文件  
zhengfei.tmp.2:匹配的结果  

例如:这是我匹配到的其中一条数据
![这是我匹配到的其中一条数据]({{site.baseurl}}/assets/img/2015-10-12/2015-10-12 15:49:39.png)

当我匹配到这些数据后,我想匹配ID为5356的数据
![这是我匹配到的其中一条数据]({{site.baseurl}}/assets/img/2015-10-12/2015-10-12 16:07:51.png)

\-A:向下打印多少行  
\-B:向上打印多少行  

#总结

对于此次日志分析,otter同步没有问题(这么牛逼的东东,怎能出现这种问题),可以判断是业务方那边出了问题.

对于各自可以根据自己的需求做出灵活的应用,祝你好运!!!


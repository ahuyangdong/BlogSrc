---
title: MyBatis数据库连接出现Communications link failure现象的解决办法（dbcp2连接池）
date: 2017-11-16 12:43:35
id: 124335
tags: [MyBatis,dbcp2,数据库]
categories: Java Web
---
## 背景 ##
近期项目生产环境中老是出现"Communications link failure，The last packet successfully received from the server was ** millisecond ago."
然后系统就无法读取数据库了。

解决办法
----

1、排查mysql数据库配置文件my.cnf中有无wait_timeout、interactive_timeout两个参数，要添加参数为

``` java
wait_timeout=864000
interactive_timeout=864000
```
864000为10天，具体的大小可根据系统情况自行调整。修改完后重启数据库服务生效。

2、检查web应用中的数据库连接池配置有无问题。我这边使用的是dbcp2，新增加了配置参数：

``` xml
<!-- 指明连接是否被空闲连接回收器(如果有)进行检验 -->
<property name="testWhileIdle" value="true"/>
<!-- 运行一次空闲连接回收器的时间间隔（60秒）-->
<property name="timeBetweenEvictionRunsMillis" value="${timeBetweenEvictionRunsMillis}"/>
<!-- 验证时使用的SQL语句 -->
<property name="validationQuery" value="SELECT 1" />
<!-- 借出连接时不要测试，否则很影响性能 -->
<property name="testOnBorrow" value="false"/>
```
连接池配置全部代码（dbcp2）：

``` xml
<bean id="dataSource-base" class="org.apache.commons.dbcp2.BasicDataSource"
	destroy-method="close">
	<property name="driverClassName" value="${driver}" />
	<property name="url" value="${url}" />
	<property name="username" value="${username}" />
	<property name="password" value="${password}" />
	<!-- 初始化连接大小 -->
	<property name="initialSize" value="${initialSize}"/>
	<!-- 连接池最大数量 -->
	<property name="maxTotal" value="${maxTotal}"/>
	<!-- 连接池最大空闲 -->
	<property name="maxIdle" value="${maxIdle}"/>
	<!-- 连接池最小空闲 -->
	<property name="minIdle" value="${minIdle}"/>
	<!-- 获取连接最大等待时间 -->
	<property name="maxWaitMillis" value="${maxWaitMillis}"/>
	<!-- 指明连接是否被空闲连接回收器(如果有)进行检验 -->
	<property name="testWhileIdle" value="true"/>
	<!-- 运行一次空闲连接回收器的时间间隔（60秒）-->
	<property name="timeBetweenEvictionRunsMillis" value="${timeBetweenEvictionRunsMillis}"/>
	<!-- 验证时使用的SQL语句 -->
	<property name="validationQuery" value="SELECT 1" />
	<!-- 借出连接时不要测试，否则很影响性能 -->
   	<property name="testOnBorrow" value="false"/>
</bean>
```
其中jdbc.properties:

``` java
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306/abc?useUnicode=true&amp;characterEncoding=utf8&amp;autoReconnect=true
username=username
password=password

#定义初始连接数  
initialSize=0
#定义最大连接数  
maxTotal=20
#定义最大空闲  
maxIdle=20
#定义最小空闲  
minIdle=1
#定义最长等待时间  
maxWaitMillis=60000
#空闲回收期运行周期（60秒）
timeBetweenEvictionRunsMillis=60000
```
配置参数修改后重新发布系统，上述问题解决了。

参考文章
----

http://blog.csdn.net/xuzhuang2008/article/details/8129204

http://blog.csdn.net/itbasketplayer/article/details/44198963

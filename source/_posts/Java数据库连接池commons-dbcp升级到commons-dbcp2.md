---
title: Java数据库连接池commons-dbcp升级到commons-dbcp2
date: 2017-11-16 18:20:29
id: 182029
tags: [数据库,连接池,dbcp2]
categories: Java Web
---
## 背景 ##
Spring Web系统数据库连接池使用的还是老版的commons-dbcp，打算由commons-dbcp升级到commons-dbcp2最新版。
## 步骤 ##

1、升级maven依赖。

commons-dbcp 1.2.2 升级到 commons-dbcp2 2.1.1
maven项目依赖变更，由

``` xml
<dependency>
	<groupId>commons-dbcp</groupId>
	<artifactId>commons-dbcp</artifactId>
	<version>1.2.2</version>
</dependency>
```
变更为:

``` xml
<dependency>
	<groupId>org.apache.commons</groupId>
    <artifactId>commons-dbcp2</artifactId>
    <version>2.1.1</version>
</dependency>
```

2、更换BasicDataSource和配置连接项

spring-mybatis.xml文件中的bean配置，class由 org.apache.commons.dbcp.BasicDataSource更换为org.apache.commons.dbcp2.BasicDataSource
bean配置由：

``` xml
<bean id="dataSource-base" class="org.apache.commons.dbcp.BasicDataSource"
	destroy-method="close">
	<property name="driverClassName" value="${driver}" />
	<property name="url" value="${url-test}" />
	<property name="username" value="${username-test}" />
	<property name="password" value="${password-test}" />
	<!-- 初始化连接大小 -->
	<property name="initialSize" value="${initialSize}"/>
	<!-- 连接池最大数量 -->
	<property name="maxActive" value="${maxActive}"/>
	<!-- 连接池最大空闲 -->
	<property name="maxIdle" value="${maxIdle}"/>
	<!-- 连接池最小空闲 -->
	<property name="minIdle" value="${minIdle}"/>
	<!-- 获取连接最大等待时间 -->
	<property name="maxWait" value="${maxWait}"/>
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
变更为：

``` xml
<bean id="dataSource-base" class="org.apache.commons.dbcp2.BasicDataSource"
	destroy-method="close">
	<property name="driverClassName" value="${driver}" />
	<property name="url" value="${url-test}" />
	<property name="username" value="${username-test}" />
	<property name="password" value="${password-test}" />
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
其中的jdbc.properties:

``` java
driver=com.mysql.jdbc.Driver
url-test=jdbc:mysql://localhost:3306/abc?useUnicode=true&amp;characterEncoding=utf8&amp;autoReconnect=true
username-test=username
password-test=password

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

3、连接池配置项变更说明

由于dbcp2使用的连接池pool对配置属性做了修改，需要修改连接池bean配置中的属性，涉及到的修改：

``` java
maxWait -> maxWaitMillis
maxActive -> maxTotal
```

注：dbcp2需要jdk1.7以上环境
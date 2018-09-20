---
title: Spring+SpringMVC+Mybatis（SSM）框架搭建教程（二）-创建项目
date: 2018-02-22 12:35:49
id: 123549
tags: [SpringMVC,SSM]
categories: Java Web
---
## 背景 ##
上一篇《[Spring+SpringMVC+Mybatis（SSM）框架搭建教程（一）-Maven工具配置](http://www.ahuyangdong.top/2018/02/09/123916)》已将框架搭建所需的Maven环境配置完成，本篇我们完成MyEclipse创建Maven webapp项目+Spring框架依赖导入。
## 创建项目
1、项目使用MyEclipse10.0开发，通过File->New->Project，选择Maven Project，点击Next。
![这里写图片描述](https://img-blog.csdn.net/20180212162026408?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2、不要勾选"create a simple..."，选择好项目存储位置后，点击Newxt。
![这里写图片描述](https://img-blog.csdn.net/2018021216211678?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
3、在中间的表格栏选中：maven-archetype-webapp，点击Next。
![这里写图片描述](https://img-blog.csdn.net/20180212162255955?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
4、输入项目的基本信息，包括组织ID、项目名称、包名等，点击Finish。
![这里写图片描述](https://img-blog.csdn.net/2018021216244323?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
5、修改项目文件夹名称（可选步骤）。
新建的项目生成时，名称上会带上Maven Webapp字样，如果觉得不需要可以给改掉。在项目上右键Refactor-Rename，去掉“ Maven Webapp”就行了。
## 运行项目
新建好的web项目是一个很基础的目录结构，目录如下：
![这里写图片描述](https://img-blog.csdn.net/20180212162931702?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
没有java源码目录，我们需要自己建一个目录。在项目上右键->New->Source Folder，输入“src/main/java”，这个目录是后面我们放置java源码的。目录结构变成：
![这里写图片描述](https://img-blog.csdn.net/20180212165911595?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
此时的项目已经可以运行，我们部署到tomcat中访问http://localhost:8080/SSMDemo/，就能看到首页信息了。
![这里写图片描述](https://img-blog.csdn.net/20180212163018326?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 框架基础配置 ##
1、导入框架依赖。
由于搭建的是Spring+SpringMVC+Mybatis框架，我们分项列出基本的包依赖。其他情况根据需要来添加。

 - spring框架包
 - springMVC包
 - mybatis框架包
 - mybatis与spring集成包
 - javaee包
 - mysql 驱动包
 - log4j日志包
 - json相关包
 - common常用组件包
 - 其他需要的包

我们需要在项目的Pom.xml中配置好所需的依赖，Pom里面的语法可以自行查阅。完整的Pom文件：

``` xml
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<groupId>com.dommy</groupId>
	<artifactId>SSMDemo</artifactId>
	<packaging>war</packaging>
	<version>0.0.1-SNAPSHOT</version>
	<name>SSMDemo</name>
	<url>http://maven.apache.org</url>

	<properties>
		<!-- 文件编码 -->
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<!-- spring版本号 -->
		<spring.version>4.3.12.RELEASE</spring.version>
		<!-- mybatis版本号 -->
		<mybatis.version>3.2.6</mybatis.version>
		<!-- log4j日志文件管理包版本 -->
		<slf4j.version>1.7.7</slf4j.version>
		<log4j.version>1.2.17</log4j.version>
	</properties>

	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>3.8.1</version>
			<scope>test</scope>
		</dependency>

		<!-- spring核心包 -->
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-core</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-web</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-oxm</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-tx</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-jdbc</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-webmvc</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-aop</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-context-support</artifactId>
			<version>${spring.version}</version>
		</dependency>

		<dependency>
			<groupId>org.springframework</groupId>
			<artifactId>spring-test</artifactId>
			<version>${spring.version}</version>
		</dependency>
		<!-- mybatis核心包 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis</artifactId>
			<version>${mybatis.version}</version>
		</dependency>
		<!-- mybatis/spring包 -->
		<dependency>
			<groupId>org.mybatis</groupId>
			<artifactId>mybatis-spring</artifactId>
			<version>1.2.2</version>
		</dependency>
		<!-- 导入java ee jar 包 -->
		<dependency>
			<groupId>javax</groupId>
			<artifactId>javaee-api</artifactId>
			<version>7.0</version>
		</dependency>
		<!-- 导入Mysql数据库连接jar包 -->
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<version>5.1.30</version>
		</dependency>
		<!-- dbcp数据库连接池 -->
		<dependency>
			<groupId>org.apache.commons</groupId>
			<artifactId>commons-dbcp2</artifactId>
			<version>2.1.1</version>
		</dependency>
		<!-- JSTL标签类 -->
		<dependency>
			<groupId>jstl</groupId>
			<artifactId>jstl</artifactId>
			<version>1.2</version>
		</dependency>
		<!-- 日志文件管理包 -->
		<!-- log start -->
		<dependency>
			<groupId>log4j</groupId>
			<artifactId>log4j</artifactId>
			<version>${log4j.version}</version>
		</dependency>

		<!-- 格式化对象，方便输出日志 -->
		<dependency>
			<groupId>com.alibaba</groupId>
			<artifactId>fastjson</artifactId>
			<version>1.2.23</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>${slf4j.version}</version>
		</dependency>

		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-log4j12</artifactId>
			<version>${slf4j.version}</version>
		</dependency>
		<!-- log end -->
		<!-- 映入JSON -->
		<dependency>
			<groupId>com.fasterxml.jackson.core</groupId>
			<artifactId>jackson-databind</artifactId>
			<version>2.5.3</version>
		</dependency>
		<!-- 上传组件包 -->
		<dependency>
			<groupId>commons-fileupload</groupId>
			<artifactId>commons-fileupload</artifactId>
			<version>1.3.1</version>
		</dependency>
		<dependency>
			<groupId>commons-io</groupId>
			<artifactId>commons-io</artifactId>
			<version>2.4</version>
		</dependency>
		<dependency>
			<groupId>commons-codec</groupId>
			<artifactId>commons-codec</artifactId>
			<version>1.9</version>
		</dependency>

		<!-- Gson json解析 -->
		<dependency>
			<groupId>com.google.code.gson</groupId>
			<artifactId>gson</artifactId>
			<version>2.8.0</version>
		</dependency>

		<dependency>
			<groupId>org.apache.ant</groupId>
			<artifactId>ant</artifactId>
			<version>1.8.4</version>
		</dependency>

	</dependencies>
	<build>
		<finalName>SSMDemo</finalName>
	</build>
</project>

```
2、配置web.xml，设置请求的处理机制，mybatis参数注入等。基础版的web.xml：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	
	<display-name>SSMDemo</display-name>
	
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring-mybatis.xml</param-value>
	</context-param>

	<!-- 配置SpringMVC -->
	<servlet>
		<servlet-name>SpringMVC</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:spring-mvc.xml</param-value>
		</init-param>
		<load-on-startup>1</load-on-startup>
		<async-supported>true</async-supported>
	</servlet>

	<!-- 所有请求转入MVC控制 -->
	<servlet-mapping>
		<servlet-name>SpringMVC</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>

	<!-- 设置监听器 -->
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
	<listener>
		<listener-class>org.springframework.web.util.IntrospectorCleanupListener</listener-class>
	</listener>


	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>
</web-app>
```
3、配置spring-mvc.xml。在src/main/resources下面新建文件spring-mvc.xml，填入内容：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans  
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd  
                        http://www.springframework.org/schema/context  
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd  
                        http://www.springframework.org/schema/mvc  
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">


	<!--避免IE执行AJAX时，返回JSON出现下载文件 -->
	<bean id="mappingJackson2HttpMessageConverter"
		class="org.springframework.http.converter.json.MappingJackson2HttpMessageConverter">
		<property name="supportedMediaTypes">
			<list>
				<value>text/html;charset=UTF-8</value>
			</list>
		</property>
	</bean>

	<!-- 启动SpringMVC的注解功能，完成请求和注解POJO的映射 -->
	<bean
		class="org.springframework.web.servlet.mvc.annotation.AnnotationMethodHandlerAdapter">
		<property name="messageConverters">
			<list>
				<ref bean="mappingJackson2HttpMessageConverter" />	
			</list>
		</property>
	</bean>

	<!-- 自动注册DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter 两个bean,是spring MVC为@Controllers分发请求所必须的-->
	<mvc:annotation-driven />

	<!-- 静态资源路径配置 -->
	<mvc:resources mapping="/resources/**" location="/resources/" />

	<!-- 定义跳转的文件的前后缀 ，视图模式配置 -->
	<bean
		class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<!-- 视图前缀 -->
		<property name="prefix" value="/pages/" />
		<!-- 视图后缀 -->
		<property name="suffix" value=".jsp" />
	</bean>

	<!-- 配置文件上传，如果没有使用文件上传可以不用配置，当然如果不配，那么配置文件中也不必引入上传组件包 -->
	<bean id="multipartResolver"
		class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<!-- 默认编码 -->
		<property name="defaultEncoding" value="utf-8" />
		<!-- 文件大小最大值 -->
		<property name="maxUploadSize" value="10485760000" />
		<!-- 内存中的最大值，写大时小文件不会生产临时文件，影响同步 -->
		<property name="maxInMemorySize" value="1" />
	</bean>
</beans>
```
里面的具体内容后面再说了，先这么配上。

4、配置spring-mybatis.xml，这里主要是配置数据源和相应的事务控制器等，暂时用不到。先放一个空的内容：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mvc="http://www.springframework.org/schema/mvc"
	xsi:schemaLocation="http://www.springframework.org/schema/beans    
                        http://www.springframework.org/schema/beans/spring-beans-3.1.xsd    
                        http://www.springframework.org/schema/context    
                        http://www.springframework.org/schema/context/spring-context-3.1.xsd    
                        http://www.springframework.org/schema/mvc    
                        http://www.springframework.org/schema/mvc/spring-mvc-4.0.xsd">
	

</beans>
```
## 运行项目（第二次） ##
至此，我们已经完成了依赖和基本配置项的编写，可以尝试再次运行项目，看看有没有问题。
运行前，我们需要在项目上右键->Run As->8 Maven install，让Maven下载远程jar到本地磁盘，然后导入引用至项目（自动完成）。控制台提示BUILD SUCCESS之后，我们把项目再次部署到tomcat并运行，没有报错。访问http://localhost:8080/SSMDemo/，还是能看到首页信息。
![这里写图片描述](https://img-blog.csdn.net/20180212163018326?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
如果你在配置过程中有问题，请对照上述源码部分找原因。
## 结语 ##
本篇从创建项目、依赖导入的角度讲解了框架搭建过程中的项目初始化部分，涉及注解、应用代码封装、统一化等在下一篇进行项目。 
## 源码下载

GitHub地址：
https://github.com/ahuyangdong/SSMDemo
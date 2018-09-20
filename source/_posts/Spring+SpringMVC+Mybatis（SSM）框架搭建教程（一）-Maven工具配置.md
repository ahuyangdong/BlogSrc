---
title: Spring+SpringMVC+Mybatis（SSM）框架搭建教程（一）-Maven工具配置
date: 2018-02-09 12:39:16
id: 123916
tags: [SpringMVC,SSM,Maven]
categories: Java Web
---
## 背景 ##
Spring+SpringMVC+Mybatis（SSM）框架的搭建过程中会用到Spring系列的n多个jar包，按以往依赖jar的笨办法再来添加依赖，会比较麻烦，也不利于管理。所以在这套框架搭建里面我们使用Maven构建工具来管理jar包。
## Maven简介 ##
Maven和Gradle构建工具这几年比较火，到处都能看到。构建工具主要的作用，在我理解，是导入jar、引入关联的依赖（某jar依赖的其他jar）、解决依赖冲突等，其他深入应用、原因也没有去深挖，够用就好。
## 下载 ##
下载地址：https://maven.apache.org/download.cgi
![Maven下载](https://img-blog.csdn.net/20180209101053825?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
由于本地环境是windows，我下载了apache-maven-3.3.9-bin.zip版本，最新版持续在更新，当前是apache-maven-3.5.2-bin.zip。
## 本地安装 ##
和tomcat一样，Maven也是解压安装，建议解压到D/E盘的根目录，避免路径出现空格。我这里解压到了E盘，路径为：E:\apache-maven-3.3.9，以下变量值路径也是根据个人情况来修改。
## 环境变量配置##
1、计算机->右键，属性->高级系统设置->环境变量。
![环境变量界面](https://img-blog.csdn.net/20180209101356943?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2、新建系统变量“MAVEN_HOME”、填入值“E:\apache-maven-3.3.9”
![MAVEN_HOME](https://img-blog.csdn.net/20180209101552542?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
3、新建系统变量“M2_HOME”，填入值“E:\apache-maven-3.3.9”
![M2_HOME](https://img-blog.csdn.net/20180209101658466?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
4、在系统变量Path中加入：

```
%MAVEN_HOME%\bin;
```
注意将本字符串添加到Path的最前面，结尾的分号不要丢失。
![Path配置](https://img-blog.csdn.net/20180209101841329?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
5、查看配置是否成功。
在命令行中输入mvn -version，查看Maven是否安装配置成功。
![校验maven安装情况](https://img-blog.csdn.net/20180209101935726?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## Maven settings配置##
下载的Maven包中settings.xml需要微调下，以方便我们使用。配置文件路径：E:\apache-maven-3.3.9\conf，找到settings.xml文件，复制一份到E:\MavenRepository下面（我在E盘新建了MavenRepository文件夹，用来存放Maven下载过来的jar包和Maven配置），作如下修改：

1、修改localRepository的值为指定路径，在其中新建了maven_jar，用来存放maven下载过来的jar包等。

``` xml
<localRepository>E:/MavenRepository/maven_jar</localRepository>
```
2、添加阿里云仓库镜像地址，在我们下载jar包的时候可以优先从国内镜像走，下载速度较快。

``` xml
<mirror>
     <id>alimaven</id>
     <name>aliyun maven</name>
     <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
     <mirrorOf>central</mirrorOf>        
</mirror>
```
其他配置按需修改，这里暂时只有改到这两个地方，完整的settings.xml：

``` xml
<?xml version="1.0" encoding="UTF-8"?>

<!--
Licensed to the Apache Software Foundation (ASF) under one
...
-->

<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">

  <localRepository>E:/MavenRepository/maven_jar</localRepository>
  
  <pluginGroups>
  </pluginGroups>

  <proxies>
  </proxies>

  <servers>     
  </servers>

  <mirrors>	 
	<mirror>
      <id>alimaven</id>
      <name>aliyun maven</name>
      <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
      <mirrorOf>central</mirrorOf>        
	</mirror>
  </mirrors>

  
  <profiles>
    
  </profiles>


</settings>

```
（很多默认注释我都给删掉了，篇幅有限，想了解的看原版英文注释）

## MyEclipse安装Maven插件 ##
本地配置好的Maven主要是为IDE开发编辑器使用的，我这边用的是MyEclipse10.0，受外部环境影响，没有选用Intellij Idea。所以这里讲下Maven与MyEclipse10.0的集成。

1、配置Maven路径
打开MyEclipse，进入Windows->Preferences->MyEclipse->Maven4MyEclipse->Installations。点击Add添加E盘的Maven工具。如下图：
![添加Maven工具](https://img-blog.csdn.net/20180209103840251?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
点击OK，保存即可。

2、配置用户参数
进入Windows->Preferences->MyEclipse->Maven4MyEclipse->UserSettings。点击右侧的“Browse”，选择位置：“E:\MavenRepository\settings.xml”。
如下图：
![配置用户参数](https://img-blog.csdn.net/20180209103951551?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
点击OK，保存即可。

小提示：
如果运行Maven时出现M2_HOME未识别的问题，需要在jdk的配置项目上面添加参数：

``` java
-Dmaven.multiModuleProjectDirectory=$M2_HOME
```
![M2_HOME配置](https://img-blog.csdn.net/20180209104150931?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 结语 ##
本篇将Spring+SpringMVC+Mybatis（SSM）框架搭建前夕的Maven构建工具本地配置、与MyEclipse集成方法全部说完了，下一篇开始讲框架项目新建与基本配置。
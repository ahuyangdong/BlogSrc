---
title: Spring+SpringMVC+Mybatis（SSM）框架搭建教程（四）-应用功能开发实例
date: 2018-02-28 08:31:07
id: 083107
tags: [SpringMVC,SSM]
categories: Java Web
---
## 背景
上一篇《[Spring+SpringMVC+Mybatis（SSM）框架搭建教程（三）-配置文件详解](http://www.ahuyangdong.top/2018/02/25/175821)》着重介绍了框架整合过程中的关键配置项目。本篇着重介绍此框架在应用开发过程中的实例与技巧，并给出基本的Controller层封装方法。
## 需求分析
假设我们需要利用本框架做一个学生信息呈现系统，主要包括数据库列表信息呈现和数据总量统计，需要用到页面渲染、ajax请求等技术。考虑到实际的信息系统开发场景，将两种请求方式均涉及到，分解需求为：

 - 通过SpringMVC控制器实现请求处理
 - 通过返回jsp视图页面呈现学生信息列表
 - 通过Ajax异步技术获取学生信息总量

## 新建数据库
新建mysql数据库ssm_demo，并新建表t_student。建表脚本：

``` sql
CREATE TABLE `t_student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(512) DEFAULT NULL COMMENT '姓名',
  `age` int(3) DEFAULT NULL COMMENT '年龄',
  `sex` int(1) DEFAULT NULL COMMENT '性别：1-男 2-女',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8;
```
## 应用功能代码编写
我们按照自底向上，逐级被调用的形式展开来说。
1、entity类
在com.dommy.entity包下新建Student.java类文件：

``` java
package com.dommy.entity;

/**
 * 学生实体
 */
public class Student {
	private int id; // 主键
	private String name; // 姓名
	private int age; // 年龄
	private int sex; // 性别 1-男 2-女

	public int getId() {
		return id;
	}

	public void setId(int id) {
		this.id = id;
	}

	public String getName() {
		return name;
	}

	public void setName(String name) {
		this.name = name;
	}

	public int getAge() {
		return age;
	}

	public void setAge(int age) {
		this.age = age;
	}

	public int getSex() {
		return sex;
	}

	public void setSex(int sex) {
		this.sex = sex;
	}
}

```
2、mapper映射文件
mapper映射即为sql脚本文件，结合Mybatis框架，在com.dommy.mapper包下新建StudentMapper.xml：

``` xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper PUBLIC "-//ibatis.apache.org//DTD Mapper 3.0//EN"      
 "http://mybatis.apache.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.dommy.dao.StudentDao">

	<!-- 查询列表 -->
	<select id="getList" parameterType="Map" resultType="Student">
		select * from t_student
		where 1=1
		<if test="name!=null and name!=''"> and name like CONCAT('%',#{name},'%') </if>
		order by id
	</select>

	<!-- 查询数量 -->
	<select id="getCount" parameterType="Map" resultType="int">
		select count(*) from t_student
		where 1=1
		<if test="name!=null and name!=''"> and name like CONCAT('%',#{name},'%') </if>
	</select>
</mapper>
```
其中的resultType使用到了别名，在src/main/resources里面的sqlMapConfig.xml里面配置：

``` xml
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration PUBLIC "-//ibatis.apache.org//DTD Config 3.0//EN" 
	"http://ibatis.apache.org/dtd/ibatis-3-config.dtd">
	
<configuration>
	<!-- sql日志 -->
	<settings>  
        <setting name="logImpl" value="LOG4J"/>  
    </settings> 
    
	<!-- 别名 -->
	<typeAliases>
		<typeAlias type="com.dommy.entity.Student" alias="Student"/>
	</typeAliases>
</configuration>

```
3、Dao层接口
Mybatis接口方法层，主要是方法声明，方法名与Mapper中对应的id一致。（Mybatis语法与用法这里不说）
在com.dommy.dao包下新建接口StudentDao.java：

``` java
package com.dommy.dao;

import java.util.List;
import java.util.Map;

import com.dommy.entity.Student;

/**
 * 学生信息dao层接口
 */
public interface StudentDao {

	/**
	 * 读取学生列表
	 * @param param 查询参数
	 * @return List<Student> 列表
	 */
	List<Student> getList(Map<String, Object> param);

	/**
	 * 读取学生数量
	 * @param param 查询参数
	 * @return int 数量
	 */
	int getCount(Map<String, Object> param);
}

```
*注意：dao接口文件中只需要写方法声明，不需要写任何与Spring相关的注解。

4、Service接口层
Service层我们习惯定义接口，所以这里也保留这种形式，在com.dommy.service包下定义StudentService.java：

``` java
package com.dommy.service;

import java.util.List;

import com.dommy.entity.Student;

/**
 * 学生信息业务层接口
 */
public interface StudentService {
	
	/**
	 * 读取学生列表
	 * @param name 姓名
	 * @return List<Student> 列表
	 */
	List<Student> getList(String name);

	/**
	 * 读取学生数量
	 * @param name 姓名
	 * @return int 数量
	 */
	int getCount(String name);
}

```
*注意：Service接口文件中只需要写方法声明，不需要写任何与Spring相关的注解。

5、Service实现类
在 com.dommy.service.impl包下新建StudentServiceImpl.java来实现StudentService接口：

``` java
package com.dommy.service.impl;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.annotation.Resource;

import org.springframework.stereotype.Service;

import com.dommy.dao.StudentDao;
import com.dommy.entity.Student;
import com.dommy.service.StudentService;

@Service
public class StudentServiceImpl implements StudentService {

	@Resource
	StudentDao dao;

	@Override
	public List<Student> getList(String name) {
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("name", name);
		return dao.getList(param);
	}

	@Override
	public int getCount(String name) {
		Map<String, Object> param = new HashMap<String, Object>();
		param.put("name", name);
		return dao.getCount(param);
	}

}

```
这里做两点说明：

 - StudentServiceImpl类上需要加@Service注解，让Spring知道它是Service类，在其引用处注入对象；
 - Service内部调用dao层时，如StudentDao对象，其引用处上方需要加@Resource或@Autowired注解，让Spring知道它需要被注入。至于是用@Resource还是@Autowired，没有去深究，表面上看一个是java提供的，一个是Spring框架提供的。
 
6、Controller交互层
Controller是整个请求处理过程中的桥梁，可以理解为网络请求进入Controller并拿到Controller给的返回内容后结束（实际比这个复杂，只是通俗理解）。Controller用到的方法比较容易封装起来，后文单独说封装的事情。
这里先直接写结果了，在com.dommy.controller.student包下新建StudentController.java：

``` java
package com.dommy.controller.student;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import com.dommy.controller.BaseController;
import com.dommy.controller.MVBuilder;
import com.dommy.entity.Student;
import com.dommy.service.StudentService;

/**
 * 学生信息交互层
 */
@Controller
@RequestMapping("/student")
public class StudentController extends BaseController {

	@Resource
	StudentService studentService; // 注入student业务类接口

	/**
	 * 查询学生信息列表
	 * <p>可按学生信息模糊查询</p>
	 * @param name 学生姓名
	 * @return ModelAndView 界面视图
	 */
	@RequestMapping("/list")
	public ModelAndView list(String name) {
		MVBuilder mvb = getMVBuilder();
		try {
			List<Student> students = studentService.getList(name);
			mvb.addAttribute("students", students);
			mvb.addAttribute("name", name); // 查询参数返回
		} catch (Exception e) {
			e.printStackTrace();
		}
		return forward("student", mvb);
	}

	/**
	 * 查询学生信息总数
	 * <p>这是一个接收异步请求并输出结果的示例</p>
	 * @param response
	 * @param name 学生姓名
	 */
	@RequestMapping("/getCount")
	public void getCount(HttpServletResponse response, String name) {
		Map<String, Object> map = new HashMap<String, Object>();
		String result = ERROR;
		try {
			int count = studentService.getCount(name);
			map.put("count", count);
			result = SUCCESS;
		} catch (Exception e) {
			e.printStackTrace();
		}
		map.put("result", result);
		this.renderJson(response, map);
	}
}

```
代码说明

 - @Controller注解告诉Spring这个bean需要纳入管理，Controller交给Spring管理时可以用注解@Scope设置单例、多实例等。我们一般都用单例，默认值不用改；
 - @RequestMapping("/student")注解告诉SpringMVC框架凡是路径上匹配/student的请求，转入本类处理；
 - StudentService同样采用@Resource注解注入对象；
 - list方法上加了@RequestMapping("/list")标识路径上在/student之后如果是/list，则转入本方法处理；
 - forward("student", mvb)方法是封装的，“student”字符串标识视图名称，结合spring-mvc.xml配置，对应的视图路径是pages/student.jsp。
 - getCount方法中的renderJson是输出json的封装方法，后文来说。

7、jsp视图层
在webapp/pages目录下新建student.jsp代码，编写内容：

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib uri="http://java.sun.com/jsp/jstl/core" prefix="c" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/fmt" prefix="fmt" %>
<%
	String path = request.getContextPath();
	String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE html>
<html lang="zh-CN">
  	<head>
    	<base href="<%=basePath%>">
    	<title>学生信息</title>
    	<meta charset="utf-8">
		<meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
		<meta name="viewport" content="width=device-width, initial-scale=1">
		<script src="resources/js/jquery-1.9.1.min.js"></script>
  	</head>
<body>
  	<div>
  		<div>
			<span>学生管理</span>
		</div>
  		<div>
			<form id="frm" method="post">
				<span>姓名：</span>
				<input type="text" id="name" name="name" value="${name }"/>
				&ensp;
				<button type="button" onclick="query();">查询</button>
			</form>
 			<br/>
	  	</div>
		<div>
			<table>
				<tr>
					<th>序号</th>
					<th>姓名</th>
					<th>年龄</th>
					<th>性别</th>
				</tr>
				<c:forEach var="student" items="${students }" varStatus="i">
					<tr>
					 	<td>${i.count+(page.pageIndex-1)*page.pageSize }</td>
						<td>${student.name }</td>
						<td>${student.age}</td>
						<td>
							<c:if test="${student.sex==1 }">男</c:if>
							<c:if test="${student.sex==2 }">女</c:if>
						</td>
					</tr>
				 </c:forEach>
			</table>
			<br/>
			总数：<span id="count"></span>
		</div>
	</div>
</body>
<script>
$(function() {
	// 页面加载完成后异步请求count值
	getCount();
});

// 查询
function query() {
	$("#frm").attr("action", "<%=basePath%>student/list");
	$("#frm").submit();
}

// ajax异步请求
function getCount() {
	$.ajax({
		type: "post",
		url: "<%=basePath %>student/getCount",
		data: {
			"name": $("#name").val()
		},
		dataType: "json",
		success: function(data) {
			if (data.result == "success") {
				$("#count").html(data.count);
			} else if (data == "error") {
				$("#count").html("");
				alert("ajax查询失败！");
			} else {
				$("#count").html("");
			}
		}
	});
}
</script>
</html>

```
8、项目结构
项目代码基本编写完成，展示一下项目结构：
![完整项目结构](https://img-blog.csdn.net/2018022217420738?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 预览功能
浏览器中输入：http://localhost:8080/SSMDemo/student/list访问后，呈现效果：
![项目预览情况](https://img-blog.csdn.net/20180222174027137?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## Controller封装
Controller层因为用到参数传递、输入输出等方法，可封装性较强。封装了一个BaseController作为基类供交互层继承，并编写了一个MVBuilder作为ModelAndView构造器，方便开发时调用。
1、BaseController.java类
``` java
package com.dommy.controller;

import java.io.PrintWriter;
import java.net.InetAddress;
import java.net.UnknownHostException;

import javax.annotation.Resource;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.web.servlet.ModelAndView;

import com.google.gson.Gson;

/**
 * 交互层基类
 */
public class BaseController {
	protected static final String SUCCESS = "success";
	protected static final String ERROR = "error";

	@Resource
	private HttpServletRequest request;

	/**
	 * 获取模型视图构造器
	 * @return MVBuilder 模型视图构造器
	 */
	protected MVBuilder getMVBuilder() {
		return new MVBuilder();
	}

	/**
	 * 跳转到指定页面
	 * @param viewName 视图名称
	 * @param mvb 视图构造器
	 * @return ModelAndView 模型视图对象
	 */
	protected ModelAndView forward(String viewName, MVBuilder mvb) {
		mvb.getMV().setViewName(viewName);
		return mvb.getMV();
	}

	/**
	 * 输出文本内容到页面上
	 * @param text 文本内容
	 * @param contentType 返回类型
	 */
	private void render(HttpServletResponse response, String text, String contentType) {
		if (response != null) {
			response.setHeader("Pragma", "No-cache");
			response.setHeader("Cache-Control", "no-cache");
			response.setDateHeader("Expires", 0);
			response.setContentType(contentType);
			PrintWriter pWriter = null;
			try {
				pWriter = response.getWriter();
				pWriter.write(text);
				pWriter.flush();
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				if (pWriter != null) {
					pWriter.close();
				}
			}
		}
	}

	/**
	 * 输出普通文本到页面
	 * @param text 普通文本
	 */
	protected void renderText(HttpServletResponse response, String text) {
		this.render(response, text, "text/plain;charset=UTF-8");
	}

	/**
	 * 输出Html格式的文本到页面
	 * @param text Html文本内容
	 */
	protected void renderHtml(HttpServletResponse response, String html) {
		this.render(response, html, "text/html;charset=UTF-8");
	}

	/**
	 * 输出Json格式的文本到页面
	 * @param json Json文本内容
	 */
	protected void renderJson(HttpServletResponse response, String json) {
		this.render(response, json, "text/json;charset=UTF-8");
	}

	/**
	 * 输出Json格式的文本到页面
	 * @param obj 待转换对象
	 */
	protected void renderJson(HttpServletResponse response, Object obj) {
		Gson gson = new Gson();
		String json = gson.toJson(obj);
		this.renderJson(response, json);
	}

	/**
	 * 输出XML格式的文本到页面
	 * @param xml XML文本内容
	 */
	protected void renderXML(HttpServletResponse response, String xml) {
		this.render(response, xml, "text/xml;charset=UTF-8");
	}

	/**
	 * 输出结果“success”到页面
	 */
	protected void renderSuccess(HttpServletResponse response) {
		this.renderText(response, SUCCESS);
	}

	/**
	 * 输出结果“error”到页面
	 */
	protected void renderError(HttpServletResponse response) {
		this.renderText(response, ERROR);
	}

	/**
	 * 获取访问者IP
	 * @return String IP地址
	 */
	protected String getIp() {
		String ip = getRequest().getHeader("x-forwarded-for");
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = getRequest().getHeader("Proxy-Client-IP");
		}
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = getRequest().getHeader("WL-Proxy-Client-IP");
		}
		if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip)) {
			ip = getRequest().getRemoteAddr();
		}
		if (ip.equals("0:0:0:0:0:0:0:1") || ip.equals("127.0.0.1")) {
			InetAddress addr;
			try {
				addr = InetAddress.getLocalHost();
				ip = addr.getHostAddress().toString();// 获得本机IP
			} catch (UnknownHostException e) {
				e.printStackTrace();
			}
		}
		return ip;
	}

	/**
	 * 获取request对象
	 * @return HttpServletRequest
	 */
	protected HttpServletRequest getRequest() {
		return request;
	}

}

```
2、MVBuilder.java类

``` java
package com.dommy.controller;

import org.springframework.web.servlet.ModelAndView;

/**
 * 视图模型构造器
 */
public class MVBuilder {
	private ModelAndView mv;

	public MVBuilder() {
		mv = new ModelAndView();
	}

	/**
	 * 添加界面参数
	 * @param attributeName 参数名称
	 * @param attributeValue 参数值
	 */
	public void addAttribute(String attributeName, Object attributeValue) {
		mv.addObject(attributeName, attributeValue);
	}

	/**
	 * 获取模型视图
	 * @return ModelAndView
	 */
	public ModelAndView getMV() {
		return mv;
	}
}

```
## 结语
本篇通过一个学生信息查询功能的实现，解释了如果使用前面搭建出来的SSM框架。通过列表查询和数量查询两个需求展现了页面视图、异步数据响应两种形态的请求方式。通用的信息系统实现技术多可以以此类推。

下一篇通过讲解字符编码过滤器、空格过滤器对本框架的丰富性进行扩展。
## 源码下载

GitHub地址：
https://github.com/ahuyangdong/SSMDemo
---
title: Spring+SpringMVC+Mybatis（SSM）框架搭建教程（五）-扩展：过滤器Filter应用
date: 2018-03-06 08:35:02
id: 083502
tags: [SpringMVC,SSM]
categories: Java Web
---
## 背景
上一篇《[Spring+SpringMVC+Mybatis（SSM）框架搭建教程（四）-应用功能开发实例](http://www.ahuyangdong.top/2018/02/28/083107)》着重介绍了框架在项目开发过程中的使用方法，以实例的方式讲解了两种请求方式的代码编写形式。本篇着重介绍此框架在应用开发过程中的扩展——过滤器的配置。
## 目标
本篇我们要实现两种过滤器：

 - 字符编码过滤器
 - 参数空格过滤器 

## 字符编码过滤器
在中文软件系统中，中文汉字在不同的编码形式下呈现形态不一，可能会出现乱码，一般来说，程序中统一对汉字以“UTF-8”编码形式表示。在未设置字符编码过滤器时，学生信息查询功能（接上一篇开发的示例系统）中输入姓名：张三，点击查询后，得到下图内容
![乱码](https://img-blog.csdn.net/20180226122703305?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
我们可以看到，原来的“张三”，从后台过了一下，显示回前台是已经变了样，呈现为乱码，主要问题就是框架层面并未对字符编码做处理，默认使用ISO-8859-1编码集，此时中文就出现了乱码问题。
处理此问题可分为两步：
1、在项目中新建com.dommy.filter包，写一个字符编码过滤器，实现import javax.servlet.Filter接口：

``` java
package com.dommy.filter;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;
import java.util.Map.Entry;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

/**
 * 字符编码过滤器
 */
public class CharacterEncodingFilter implements Filter {
	@SuppressWarnings("unused")
	private FilterConfig filterConfig = null; // 过滤器配置参数
	private String encoding = null; // 字符编码

	public CharacterEncodingFilter() {
		super();
	}

	public void destroy() {
		this.filterConfig = null;
		this.encoding = null;
	}

	/**
	 * 过滤方法
	 */
	public void doFilter(ServletRequest req, ServletResponse res, FilterChain fc) throws IOException,
			ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		String methodName = request.getMethod();
		if ("post".equalsIgnoreCase(methodName)) {
			// post请求编码设置
			request.setCharacterEncoding(this.encoding);
		} else if ("get".equalsIgnoreCase(methodName)) {
			// get请求编码设置，需要逐个参数解析
			request = new MyRequestWrapper(request);
			HashMap map = new HashMap(request.getParameterMap());
			if (map != null && map.size() > 0) {
				Set<Entry> entrySet = map.entrySet();
				for (Map.Entry entry : entrySet) {
					Object key = entry.getKey();
					Object value = entry.getValue();
					if (value instanceof String[]) {
						String[] valueArray = (String[]) value;
						for (int i = 0; i < valueArray.length; ++i) {
							// 改编码
							valueArray[i] = doEncoding(valueArray[i]);
						}
						map.put(key, valueArray);
					}
				}
			}

			ParameterRequestWrapper wrapRequest = new ParameterRequestWrapper(request, map);
			request = wrapRequest;
		}

		// 设置输出的字符编码
		res.setContentType("text/html;charset=" + encoding);
		// 继续执行后面的程序
		fc.doFilter(request, res);
	}

	public void init(FilterConfig filterConfig) throws ServletException {

		this.filterConfig = filterConfig;

		/* 获取字符编码 */
		this.encoding = filterConfig.getInitParameter("encoding");
		if (encoding == null || encoding.length() == 0) {
			encoding = "UTF-8";
		}
	}

	/**
	 * 内部类，继承自 HttpServletRequestWrapper 类
	 */
	public class MyRequestWrapper extends HttpServletRequestWrapper {
		HttpServletRequest request = (HttpServletRequest) super.getRequest();

		public MyRequestWrapper(HttpServletRequest request) {
			super(request);
		}

		@Override
		public String getParameter(String name) {
			String value = this.encode(request.getParameter(name));
			return value;
		}

		@Override
		public String[] getParameterValues(String name) {
			String[] values = request.getParameterValues(name);
			if (values != null) {
				for (int i = 0; i < values.length; i++) {
					values[i] = this.encode(values[i]);
				}
			}
			return values;
		}

		/**
		 * 字符编码处理
		 * @param value 编码前的字符串
		 * @return 编码后的字符串
		 */
		public String encode(String value) {
			if (value != null) {
				try {
					value = new String(value.getBytes("ISO-8859-1"), CharacterEncodingFilter.this.encoding);
				} catch (UnsupportedEncodingException e) {
					e.printStackTrace();
				}
			}
			return value;
		}
	}

	/**
	 * 修改编码
	 * @param value 编码前的字符串
	 * @return String 编码后的字符串
	 */
	private String doEncoding(String value) {
		if (value != null && value.length() > 0) {
			try {
				value = new String(value.getBytes("ISO-8859-1"), "UTF-8");
			} catch (UnsupportedEncodingException e) {
				e.printStackTrace();
			}
		}
		return value;
	}
}
```
2、在web.xml文件中注册此编码器，设置要对所有的请求进行过滤。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	...
	<!-- 配置编码过滤器 -->
	<filter>
		<filter-name>encodingFilter</filter-name>
		<filter-class>com.dommy.filter.CharacterEncodingFilter</filter-class>
		<async-supported>true</async-supported>
		<init-param>
			<param-name>encoding</param-name>
			<param-value>UTF-8</param-value>
		</init-param>
	</filter>
	<filter-mapping>
		<filter-name>encodingFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
	...
</web-app>
```
重启tomcat服务器，再输入“张三”时，查询结果变成了：
![中文已转码](https://img-blog.csdn.net/20180226123334432?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
我们可以发现中文已经正确显示。

*（当然如果不设置过滤器，在每个Controller里面也可以对每个参数逐个编码解码，不过那样会很麻烦；或者在tomcat容器中设置编码也可以解决，不过会受限于容器配置，部署的时候必须保证配置统一）

## 参数空格过滤器
用户在使用系统时有意无意可能会输入空格，一般来说在字符串首尾的空格都是无意义的（密码值除外），所以常见系统里面还可以加入参数空格过滤器。同样是两步走：
1、在项目包com.dommy.filter里，写一个参数空格编码过滤器，实现import javax.servlet.Filter接口：

``` java
package com.dommy.filter;

import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.Map.Entry;
import java.util.Set;

import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.http.HttpServletRequest;

/**
 * 请求参数值空格过滤器（去除前后空格）
 */
public class ParameterTrimFilter implements Filter {
	public void init(FilterConfig filterconfig) throws ServletException {
	}

	public void destroy() {
	}

	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) throws IOException,
			ServletException {
		HttpServletRequest req = (HttpServletRequest) request;
		HashMap map = new HashMap(request.getParameterMap());
		if (map != null && map.size() > 0) {
			Set<Entry> entrySet = map.entrySet();
			// 遍历，参数值去空格
			for (Map.Entry entry : entrySet) {
				Object key = entry.getKey();
				Object value = entry.getValue();
				if (value instanceof String[]) {
					String[] valueArray = (String[]) value;
					for (int i = 0; i < valueArray.length; ++i) {
						// 去空格
						valueArray[i] = valueArray[i].trim();
					}
					map.put(key, valueArray);
				}
			}
		}

		// 类型转换
		ParameterRequestWrapper wrapRequest = new ParameterRequestWrapper(req, map);
		request = wrapRequest;
		chain.doFilter(request, response);
	}
	
}

```
其中，ParameterRequestWrapper类：

``` java
package com.dommy.filter;

import java.util.Enumeration;
import java.util.Map;
import java.util.Vector;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;

public class ParameterRequestWrapper extends HttpServletRequestWrapper {
	private Map params;

	public ParameterRequestWrapper(HttpServletRequest request, Map newParams) {
		super(request);
		this.params = newParams;
	}

	public Map getParameterMap() {
		return this.params;
	}

	public Enumeration getParameterNames() {
		Vector l = new Vector(this.params.keySet());
		return l.elements();
	}

	public String[] getParameterValues(String name) {
		Object v = this.params.get(name);
		if (v == null)
			return null;
		if (v instanceof String[])
			return (String[]) v;
		if (v instanceof String) {
			return new String[] { (String) v };
		}
		return new String[] { v.toString() };
	}

	public String getParameter(String name) {
		Object v = this.params.get(name);
		if (v == null)
			return null;
		if (v instanceof String[]) {
			String[] strArr = (String[]) v;
			if (strArr.length > 0) {
				return strArr[0];
			}
			return null;
		}
		if (v instanceof String) {
			return (String) v;
		}
		return v.toString();
	}
}
```
2、在web.xml文件中注册此编码器，设置要对所有的请求进行过滤。

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
	xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	version="3.0">
	...

	<!-- 配置去空格过滤器 -->
	<filter>
		<filter-name>ParameterTrimFilter</filter-name>
		<filter-class>com.dommy.filter.ParameterTrimFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>ParameterTrimFilter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
	...
</web-app>
```
重新发布并启动项目，在界面中输入“   张三   ”，点击查询：
![带空格的查询](https://img-blog.csdn.net/20180226123955959?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
可以看到查询结果中能查到张三，而且姓名的填入值也变成了“张三”，空格全部被过滤了。
![过滤空格的查询结果](https://img-blog.csdn.net/20180226124006458?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
## 结语
本篇从过滤器的角度对SSM框架的搭建进行了扩展，在本框架中还可以添加SessionFilter用以验证用户身份，当然还可以使用SpringMVC提供的HandlerInterceptor接口来实现请求拦截，用到的就是AOP方面的技术了。

至此，Spring+SpringMVC+Mybatis（SSM）框架搭建方案的内容已经全部讲完，下一篇将对前面的内容做一个小结，并给出示例代码的下载地址。
## 源码下载

GitHub地址：
https://github.com/ahuyangdong/SSMDemo
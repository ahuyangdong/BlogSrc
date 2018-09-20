---
title: Java Web项目漏洞：检测到目标URL存在http host头攻击漏洞解决办法
date: 2018-01-18 08:47:29
id: 084729
tags: [Web漏洞,host头攻击]
categories: Java Web
---
## 背景 ##
项目上线之后使用绿盟或Acunetix安全扫描工具扫描后发现了头攻击漏洞。截图如下：

![漏洞](https://img-blog.csdn.net/20180118083741654?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
## 漏洞提示 ##
检测工具在检测出漏洞后给予的提示为：
![漏洞解决办法](https://img-blog.csdn.net/20180118083821047?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
大意为不要使用request中的serverName，也就是说host header可能会在攻击时被篡改，依赖request的方法是不可靠的，形如JSP头部中的：

``` java
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
```
这样的使用方法就会被漏洞检测工具查出来，认定有头攻击漏洞。
## 解决办法 ##
提示中说，如果是php的话不要用SERVER_NAME，apache和Nginx通过设置虚拟机来纪要非法header，而web开发中常见的运行容器就是tomcat，网络查找出的解决方案大多不适用，最后，我们找到了一个折中的办法。
主要解决办法，就是在请求拦截上面做host合法性校验，拦截掉非法请求。

``` java
public class SessionFilter implements Filter {

	public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain) throws IOException,
			ServletException {
		HttpServletRequest request = (HttpServletRequest) req;
		HttpServletResponse response = (HttpServletResponse) res;

		// 头攻击检测
		String requestHost = request.getHeader("host");
		if (requestHost != null && !ServerWhiteListUtil.isWhite(requestHost)) {
			response.setStatus(403);
			return;
		}
		...
	}
}
```
上述代码是常见的web系统拦截器doFilter方法，我们在方法开始的地方做host判定，如果不在白名单内，则**返回403状态码**。漏洞工具收到403后认为访问请求已被终止，就不会报错了。

其中，ServerWhiteListUtil.isWhite(requestHost))方法：

``` java
package ...;

import java.io.InputStreamReader;
import java.util.List;

import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;

/**
 * 服务器白名单列表
 */
public class ServerWhiteListUtil {
	private static List<String> whiteList = null;;

	static {
		try {
			// 读取白名单列表
			whiteList = new Gson().fromJson(
					new InputStreamReader(ServerWhiteListUtil.class.getResourceAsStream("/serverWhiteList.json")),
					new TypeToken<List<String>>() {
					}.getType());
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

	/**
	 * 判断当前host是否在白名单内
	 * @param host 待查host
	 * @return boolean 是否在白名单内
	 */
	public static boolean isWhite(String host) {
		if (whiteList == null || whiteList.size() == 0) {
			return true;
		}
		for (String str : whiteList) {
			if (str != null && str.equals(host)) {
				return true;
			}
		}
		return false;
	}
}

```
配置项serverWhiteList.json文件（放置在src根目录或resource配置目录，根据项目框架来定）：

``` json
[
  "www.aaa.com:78",
  "www.bbb.com:178"
]
```
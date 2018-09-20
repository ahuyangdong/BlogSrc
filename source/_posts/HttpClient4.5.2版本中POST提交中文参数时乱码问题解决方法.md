---
title: HttpClient4.5.2版本中POST提交中文参数时乱码问题解决方法
date: 2017-01-11 10:43:23
id: 104323
tags: [HttpClient,中文乱码]
categories: Java
---
在做接口封装的时候，使用最新的HttpClient工具包来发送网络请求。

在提交中文参数内容时，遇到服务端接收数据为"???"等乱码情况，经查证和尝试，解决方法如下：

``` java
MultipartEntityBuilder mEntityBuilder = MultipartEntityBuilder.create();
		mEntityBuilder.addTextBody("appId", this.appId);
		mEntityBuilder.addTextBody("userJson", new Gson().toJson(user), ContentType.APPLICATION_JSON);
		String result = HttpUtil.httpPost(ServerURL.URL_BASE + ServerURL.URL_REGISTER, mEntityBuilder);
```

即：在包含中文的参数体上，添加第三个参数

``` java
 ContentType.APPLICATION_JSON
```

``` java
org.apache.http.entity.ContentType.APPLICATION_JSON =  = create(
            "application/json", Consts.UTF_8);
```

该方法将中文编码设定为UTF-8。



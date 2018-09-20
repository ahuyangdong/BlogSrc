---
title: Android中设置APP应用字体不缩放，文字不随系统字体大小变化
date: 2017-01-14 15:52:37
id: 155237
tags: [Android,字体大小]
categories: Android
---
应用场景
----

APP在运行时需要保持字体大小（比例）固定，按照编程设定的大小显示。当Android系统字体大小被修改时，不影响APP中文字的大小。

为什么要固定字体比例？
-----------
因为APP界面中文字元素的放大或缩小，会影响APP的呈现效果。有的时候为了界面美观和可用，我们需要做下限制，使用系统默认的字体比例

关键方法
----

在应用启动时，在Application的onCreate方法中将APP中的res配置设置为默认。见代码：

``` java
Resources res = super.getResources();
Configuration config = new Configuration();
config.setToDefaults();
res.updateConfiguration(config, res.getDisplayMetrics());
```

代码示例
----
AndroidManifest.xml中注册自定义Application
``` xml
<application
	android:name="com.xxx.application.MyApplication"
	android:allowBackup="true"
	android:icon="@drawable/ic_launcher"
	android:label="@string/app_name"
	android:theme="@style/AppTheme" 
	android:largeHeap="true">
```

MyApplication代码：

``` java
package com.xxx.application;

import android.app.Application;
import android.content.res.Configuration;
import android.content.res.Resources;

/**
 * 自定义全局Application 
 */
public class MyApplication extends Application {
	@Override
	public void onCreate() {
		super.onCreate();		
		// 加载系统默认设置，字体不随用户设置变化
		Resources res = super.getResources();
		Configuration config = new Configuration();
		config.setToDefaults();
		res.updateConfiguration(config, res.getDisplayMetrics());
	}
}
```
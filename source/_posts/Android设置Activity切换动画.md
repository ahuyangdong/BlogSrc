---
title: Android设置Activity切换动画
date: 2017-07-08 15:38:49
id: 153849
tags: [动画]
categories: Android
---
目的
--

改变Activity切换默认的动画效果。

## 方法 ##

修改Activity的theme属性。

步骤
--

 - 1、修改全局theme或自定义一个theme

``` xml
<style name="AppTheme" parent="AppBaseTheme">
       
</style>
```

 - 2、修改theme中的属性，改变动画主题。

``` xml
<style name="AppTheme" parent="AppBaseTheme">
	<item name="android:windowAnimationStyle">@style/AnimationActivity</item>
</style>
```
其中AnimationActivity可定义如下（在这里我们使用的是系统定义好的淡入淡出动画）：

``` xml
<style name="AnimationActivity" parent="@android:style/Animation.Activity">  
    <item name="android:activityOpenEnterAnimation">@android:anim/fade_in</item>  
    <item name="android:activityOpenExitAnimation">@android:anim/fade_out</item>  
    <item name="android:activityCloseEnterAnimation">@android:anim/fade_in</item>  
    <item name="android:activityCloseExitAnimation">@android:anim/fade_out</item>  
</style>
```

 - 3、在Activity声明处引用theme。

``` xml
<activity
    android:name=".MainActivity"
    android:theme="@style/AppTheme"/>
```

如果有需要可以在application中引用，定义的是全局动画方式。
![切换动画演示](https://img-blog.csdn.net/20170708161247334?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

扩展
--
上述代码中使用的动画淡入淡出是系统定义好的，fade_in.xml源码如下：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<!--
/* //device/apps/common/res/anim/fade_in.xml
**
** Copyright 2007, The Android Open Source Project
**
** Licensed under the Apache License, Version 2.0 (the "License"); 
** you may not use this file except in compliance with the License. 
** You may obtain a copy of the License at 
**
**     http://www.apache.org/licenses/LICENSE-2.0 
**
** Unless required by applicable law or agreed to in writing, software 
** distributed under the License is distributed on an "AS IS" BASIS, 
** WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. 
** See the License for the specific language governing permissions and 
** limitations under the License.
*/
-->

<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="@android:integer/config_longAnimTime"
    android:fromAlpha="0.0"
    android:interpolator="@interpolator/decelerate_quad"
    android:toAlpha="1.0" />
```
主要就是对Alpha值（透明度）做了映射，从0->1，从而完成淡入效果。
如果有需要我们也可以自己写，例如anim_activity_open_enter.xml：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<translate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="400"
    android:fromXDelta="100%p"
    android:fromYDelta="0"
    android:toXDelta="0"
    android:toYDelta="0" >

</translate>
```
通过定义转移动画，将页面从X轴最大处（界面右侧）平移至左侧，和系统默认切换方式一致。通过修改translate 动画属性可以实现自定义效果。
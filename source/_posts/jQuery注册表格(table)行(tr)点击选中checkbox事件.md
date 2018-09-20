---
title: jQuery注册表格(table)行(tr)点击选中checkbox事件
date: 2017-07-20 14:46:14
id: 144614
tags: [jQuery,table]
categories: Java Web
---
目的
--
实现鼠标点击表格行元素，就可以选中所在行内的复选框，实现数据勾选效果。多用于管理系统中数据列表上。

效果
--
录制了一个简单的动画来呈现。
![这里写图片描述](http://img.blog.csdn.net/20170720143333335?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

方法
--
这里采用jQuery框架做实现，当然原生的js也可以实现，只是麻烦一些。

``` js
$(function () {
    //除了表头（第一行）以外所有的行添加click事件.
    $("tr").slice(1).click(function () {
    	// 切换样式
    	$(this).toggleClass("tr_active");
    	// 找到checkbox对象
        var chks = $("input[type='checkbox']",this);
        var tag = $(this).attr("tag");
        if(tag=="selected"){
        	// 之前已选中，设置为未选中
        	$(this).attr("tag","");
        	chks.prop("checked",false);
        }else{
        	// 之前未选中，设置为选中
        	$(this).attr("tag","selected");
        	chks.prop("checked",true);
        }
    });
});
```

说明
--
代码比较简单，主要用到jQuery中的几个方法：

 1. slice(1)将第一行tr（一般为表头）去除绑定事件，通常表头的checkbox是用来做全选的。
 2. toggleClass为tr注册或反注册样式tr_active，tr_active是自己定义的，可根据需要修改，一般配合着是修改背景色为深色，标识为选中状态。
 3. 选择器$("input[type='checkbox']",this)在当前tr中查出checkbox。
 4. $(this).attr("tag")在tr中查找自定义标签tag。
 5. chks.prop("checked",true)修改checkbox选中状态。
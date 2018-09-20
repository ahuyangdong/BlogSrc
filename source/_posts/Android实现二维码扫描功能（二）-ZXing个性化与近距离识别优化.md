---
title: Android实现二维码扫描功能（二）-ZXing个性化与近距离识别优化
date: 2017-07-30 21:41:41
id: 314141
tags: [Zxing,二维码,识别优化]
categories: Android
---
简介
--

上一篇[Android实现二维码扫描功能（一）-ZXing插件接入](http://www.ahuyangdong.top/2017/07/30/203408 "Android实现二维码扫描功能（一）-ZXing插件接入")介绍了ZXing框架接入方法，已经可以初步集成扫码功能到项目中。

本篇我们对扫码界面进行优化，并对ZXing近距离无法识别的问题做出优化。
## 个性化定制 ##
每个APP都有自己的表现形式，实现个性化扫码界面定制，主要有两个地方：

 1. activity_scanner.xml界面文件
 2. com.google.zxing.view.ViewfinderView扫码控件


下面分别来说明。

界面文件
----
activity_scanner.xml源码如下：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <include layout="@layout/toolbar_scanner" />

    <FrameLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <SurfaceView
            android:id="@+id/scanner_view"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_gravity="center" />

        <com.google.zxing.view.ViewfinderView
            android:id="@+id/viewfinder_content"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            app:corner_color="@color/corner_color"
            app:frame_color="@color/viewfinder_frame"
            app:label_text="二维码/条形码扫描"
            app:label_text_color="@color/colorAccent"
            app:laser_color="@color/laser_color"
            app:mask_color="@color/viewfinder_mask"
            app:result_color="@color/result_view"
            app:result_point_color="@color/result_point_color" />

    </FrameLayout>

</LinearLayout>
```
界面分析：

 - 顶层采用LinearLayout实现自上至下的方向，基本不用修改；
 - `<include layout="@layout/toolbar_scanner" />`引用toolbar布局文件，稍后给出，这里可以加入菜单项目、返回键等；
 - FrameLayout布局将相机可扫描窗口叠加在一起；
 - SurfaceView装载相机内容，即相机拍摄到的画面；
 - com.google.zxing.view.ViewfinderView扫码自定义View，主要由遮罩层、四个角的边框、扫描线等组成。

toolbar_scanner.xml，比较简单

``` xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    android:background="?attr/colorPrimary"
    app:contentInsetStart="0dip">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center_vertical">

        <ImageButton
            android:id="@+id/btn_back"
            android:layout_width="40dip"
            android:layout_height="40dip"
            android:background="?attr/selectableItemBackground"
            android:padding="10dip"
            android:scaleType="centerCrop"
            android:src="@drawable/btn_back" />

        <TextView
            android:id="@+id/txt_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="扫描二维码"
            android:textColor="@android:color/white"
            android:textSize="18sp" />

    </RelativeLayout>
</android.support.v7.widget.Toolbar>

```

可根据需求调整toolbar内容，这里我们就不多说。

扫码控件定制
------
现行使用的ViewfinderView已经是经过前辈修改过的版本，他们加入了边框和扫描线，基于这个版本我们再做一些调整。

**1、提示文字的位置**

现有的提示文字是在扫描框上面的
![这里写图片描述](https://img-blog.csdn.net/20170730211553519?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
我们需要把它放置到扫描框下面，需要将源码中drawTextInfo方法调整一下：

``` java
//绘制文本
private void drawTextInfo(Canvas canvas, Rect frame) {
...
canvas.drawText(labelText, frame.left + frame.width() / 2, frame.top - CORNER_RECT_HEIGHT, paint);
}
```
调整为：

``` java
//绘制文本
private void drawTextInfo(Canvas canvas, Rect frame) {
...
canvas.drawText(labelText, frame.left + frame.width() / 2, frame.bottom + CORNER_RECT_HEIGHT * 1.5f, paint);
}
```
此处通过修改drawText的y坐标参数调整了文字位置，可以根据需要自行调整。

**2、扫码框的位置**

目前的扫码框整体位置偏下，不够美观，打算将位置往上调整一些。
修改com.google.zxing.camera.CameraManager类中的getFramingRect()方法，由

``` java
public Rect getFramingRect() {
    ...
    int leftOffset = (screenResolution.x - width) / 2;
    int topOffset = (screenResolution.y - height) / 2;
    ...
    
    return framingRect;
}
```
修改为
``` java
public Rect getFramingRect() {
    ...
    int leftOffset = (screenResolution.x - width) / 2;
    int topOffset = (screenResolution.y - height) / 3;
    ...
    
    return framingRect;
}
```
即topOffset有了减小，预览后如：
![这里写图片描述](http://img-0blog.csdn.net/20170730212515545?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

3、扫码框大小
由2我们可以看到getFramingRect()方法构造的矩形中有宽、高参数，通过修改这两个参数，可以完成扫码框大小变更。例如：

``` java
public Rect getFramingRect() {
    ...
    int width = screenResolution.x * 8 / 10;
    int height = screenResolution.y * 8 / 10;
    ...
    
    return framingRect;
}
```
则比原来的代码中放大了1/10，预览图（可对比2中图）：
![这里写图片描述](https://img-blog.csdn.net/20170730213840269?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


扫码识别优化
------

这里主要说一下近距离扫码识别不了的问题，关于扫码算法的改进涉猎不多，文尾给大家一篇文章参考。

ZXing在遇到二维码撑满扫码框的情况下识别不出结果，等多久、再聚焦都不可以，有朋友说扩大扫码框的大小可以解决这个问题，扩大后近距离扫码结果还是一致的，很难识别出来。

原因是原算法对摄像头采集的图像像素进行了裁剪，真正返回给识别算法的图像可能没有扫码框那么完整。

参考网络大神的帖子，我们将com.google.zxing.camera.CameraManager类中的buildLuminanceSource方法做了调整，不再返回剪裁后的图像。源码：

``` java
public PlanarYUVLuminanceSource buildLuminanceSource(byte[] data, int width, int height) {
    Rect rect = getFramingRectInPreview();
    int previewFormat = configManager.getPreviewFormat();
    String previewFormatString = configManager.getPreviewFormatString();
    switch (previewFormat) {
        // This is the standard Android format which all devices are REQUIRED to support.
        // In theory, it's the only one we should ever care about.
        case PixelFormat.YCbCr_420_SP:
            // This format has never been seen in the wild, but is compatible as we only care
            // about the Y channel, so allow it.
        case PixelFormat.YCbCr_422_SP:
            return new PlanarYUVLuminanceSource(data, width, height, rect.left, rect.top,
                    rect.width(), rect.height());
        default:
            // The Samsung Moment incorrectly uses this variant instead of the 'sp' version.
            // Fortunately, it too has all the Y data up front, so we can read it.
            if ("yuv420p".equals(previewFormatString)) {
                return new PlanarYUVLuminanceSource(data, width, height, rect.left, rect.top,
                        rect.width(), rect.height());
            }
    }
    throw new IllegalArgumentException("Unsupported picture format: " +
            previewFormat + '/' + previewFormatString);
}
```
修改为：

``` java
public PlanarYUVLuminanceSource buildLuminanceSource(byte[] data, int width, int height) {
    Rect rect = getFramingRectInPreview();
    int previewFormat = configManager.getPreviewFormat();
    String previewFormatString = configManager.getPreviewFormatString();
    switch (previewFormat) {
        // This is the standard Android format which all devices are REQUIRED to support.
        // In theory, it's the only one we should ever care about.
        case PixelFormat.YCbCr_420_SP:
            // This format has never been seen in the wild, but is compatible as we only care
            // about the Y channel, so allow it.
        case PixelFormat.YCbCr_422_SP:
            return new PlanarYUVLuminanceSource(data, width, height, 0, 0, width, height);
        default:
            // The Samsung Moment incorrectly uses this variant instead of the 'sp' version.
            // Fortunately, it too has all the Y data up front, so we can read it.
            if ("yuv420p".equals(previewFormatString)) {
                return new PlanarYUVLuminanceSource(data, width, height, 0, 0, width, height);
            }
    }
    throw new IllegalArgumentException("Unsupported picture format: " +
            previewFormat + '/' + previewFormatString);
}
```
主要改动的是new PlanarYUVLuminanceSource(data, width, height, 0, 0, width, height);  实际上就是将完整的相机图像内容返回，不做裁剪。

## 结语 ##
本篇对ZXing扫码工具界面定制和扫码识别优化做了讲解，解决了ZXing近距离无法识别二维码的问题。
大家也可以顺着这个思路，做更多个性化定制和优化，完善扫码功能。

## 参考资料 ##
http://iluhcm.com/2016/01/08/scan-qr-code-and-recognize-it-from-picture-fastly-using-zxing/


## 源码下载 ##

GitHub项目地址（终版）：
https://github.com/ahuyangdong/QrCodeDemo4
---
title: Android花样loading进度条（一）-水平的网页加载进度条
date: 2018-04-10 21:16:13
id: 211613
tags: [loading,网页进度条]
categories: Android
---
## 背景 ##
Android花样loading进度条系列文章主要讲解如何自定义所需的进度条，包括水平、圆形、环形、圆弧形、不规则形状等。
本篇我们从水平进度条讲起，主要是ProgressBar的水平样式应用。
## 进度条控件##
Android提供的ProgressBar控件有水平、圆形两种形态，套用不同的主题可以实现不同的大小，基本上美观一点的设计在实现的时候都需要自定义ProgressBar样式。
这里讲水平ProgressBar的样式更换方法。
## 进度条样式自定义
progressDrawable属性可为ProgressBar更换指定绘图方式。适合网页加载提示的进度条样式效果为：
![进度条效果图](https://img-blog.csdn.net/20180410210049708?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FodXlhbmdkb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
（图中红色条为进度指示条，红色背后的白色为背景色）
为此，我们写了pro_webview.xml作为progressDrawable引用的文件，内容如下：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 背景 -->
    <item android:id="@android:id/background">
        <shape>
            <solid android:color="@color/white" />
        </shape>
    </item>
    <!-- 第二进度条颜色 -->
    <item android:id="@android:id/secondaryProgress">
        <clip>
            <shape>
                <solid android:color="@color/white" />
            </shape>
        </clip>
    </item>
    <!-- 进度条 -->
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <solid android:color="@color/red_web" />
            </shape>
        </clip>
    </item>
</layer-list>
```
作几点说明：

 - @android:id/background：其对应的item表示进度条的背景色。
 - @android:id/secondaryProgress：其对应的item为第二进度条，一般表示缓冲进度条，比如音视频在播放的时候会缓冲数据，缓冲表示数据加载效果，而播放过程则使用第一进度条来显示了。第二进度条在网页加载中用不到，所以就设成白色了，和背景色一样。
 - @android:id/progress：其对应的item为进度指示条，可以理解为第一进度条，网页加载的进度在这里使用红色表示。
 - 因为进度条是随着加载情况可以动态变化的，所progress、secondaryProgress的样式设置不能使用单纯的color或shape，需要用到clip标签，因为ClipDrawable可以很好地裁剪显示（这块是Android控件实现上的事情了，开发者不用管）。

## 配置进度条属性 
将pro_webview.xml中设置的进度样式应用到ProgressBar属性中，代码如下：
``` xml
<ProgressBar
        android:id="@+id/pro_webview"
        style="?android:attr/progressBarStyleHorizontal"
        android:layout_width="match_parent"
        android:layout_height="4dip"
        android:max="100"
        android:progress="0"
        android:progressDrawable="@drawable/pro_webview"
        android:secondaryProgress="0" />
```
就可以在代码中使用设置好的ProgressBar来显示加载效果了。
## 与WebView结合
WebView在加载网页时候有接口可以计算出进度，通过不断更新ProgressBar的进度值，达到进度条随网页内容加载而动态变化的效果。动画如下：
![网页加载效果](https://img-blog.csdn.net/20180410211125173?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FodXlhbmdkb25n/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
更新ProgressBar的进度值的代码：

```
...
webView.setWebChromeClient(webChromeClient);
...

// 获取加载进度
private WebChromeClient webChromeClient = new WebChromeClient() {
    @Override
    public void onProgressChanged(WebView view, int newProgress) {
        if (newProgress == 100) {
            proWebView.setVisibility(View.GONE);
        } else {
            if (proWebView.getVisibility() == View.GONE) {
                proWebView.setVisibility(View.VISIBLE);
            }
            proWebView.setProgress(newProgress);
        }
        super.onProgressChanged(view, newProgress);
    }
};

```
通过监听网页加载newProgress来判断是否显示进度条，当达到100时隐藏进度条，未达到100时，更新进度条的指示值就可以了。
## 总结
本篇从比较简单的水平网页加载进度条入手，讲解水平进度条的使用和样式修改方法。用到的进度条ProgressBar为Android原生的控件，后续的文章将从自定义控件的角度使用Canvas来绘制进度。
## 源码下载
https://github.com/ahuyangdong/ColorfulLoading
---
title: Android花样loading进度条（二）-简单环形进度条
date: 2018-04-14 18:19:09
id: 181909
tags: [loading,环形进度条]
categories: Android
---
## 背景
Android花样loading进度条系列文章主要讲解如何自定义所需的进度条，包括水平、圆形、环形、圆弧形、不规则形状等。 
本篇我们从圆形进度条讲起，讲简单形式的环形进度条，只有进度色彩，没有进度文字，主要是使用Canvas绘制圆和圆弧。
## 效果
先上图看效果，这里有6个进度条，样式上有微妙区别，基本都属于一个类别的进度条了。
![进度条效果](/images/181909/1.gif)
6个进度条基本上分为3类：

 1. 背景条与进度条同宽度；
 2. 背景条与进度条不同宽度的，进度条略小于背景条，呈现出层次效果；
 3. 圆环式或圆饼式的效果区别。

我们以第1种作为基本的形式来讲述，需要准备的知识点有：

 - 自定义控件的坐标轴；
 - Canvas圆环、圆弧绘制方法；
 - 自定义属性；
 - Handler消息处理机制。

其中涉及到的每一个知识点都有不少内容，我们只提涉及到本次使用的知识内容。
## 知识点
1、自定义控件的坐标轴
Android系统的坐标轴示意图如下，坐标轴原点O点，可以理解为屏幕的左上角。（图片来自网络）
![这里写图片描述](/images/181909/2.jpg)
在自定义控件View的绘制过程中，O点则相当于自定义控件View区域的左上角，可以理解为View里面有一个坐标系，坐标系的原点O在本View所在位置的左上角。

2、Canvas圆环、圆弧绘制方法

1）Canvas圆环绘制方法

> android.graphics.Canvas#drawCircle(float cx, float cy, float radius, Paint paint)

此方法可绘制一个圆形，其中的参数：

 - cx：圆心O的x轴坐标；
 - cy：圆心O的y轴坐标；
 - radius：圆的半径；
 - paint：绘制图形所使用的画笔。

如果画笔使用空心模式，则绘制出一个圆环，如效果动画中的图1；如果画笔使用实心模式，则绘制出一个圆饼，如效果动画中的图3。

2）Canvas圆弧绘制方法

> android.graphics.Canvas#drawArc(RectF oval, float startAngle, float sweepAngle, boolean useCenter, Paint paint)

此方法可以绘制一个圆弧，其中：

 - oval：是一个矩形的区域，用以定义圆弧的外边缘；
 - sweepAngle：是圆弧扫过的角度；
 - userCenter：用以控制圆弧在绘画的时候，是否经过圆心；
 - paint：绘制图形所使用的画笔。
 
3、自定义属性
在自定义控件中我们免不了要用到各种颜色值、大小等参数。把这些值写死在代码里用户使用时就无法灵活控制，我们需要将属性的值定义放到用户端，以实现代码的解耦。

此时就需要一些自定义属性来支持我们对控件的属性进行定义和使用。如圆环的宽度，我们可以定义为roundWidth，值使用dimension类型。

自定义属性需要在项目res-values目录下新建或修改attrs.xml文件，具体使用方法可百度查阅，示例在下文给出。
![attrs.xml文件](/images/181909/3.png)
4、Handler消息处理机制
Android中不允许子线程（我们写个Thread运行时就算子线程）在主线程（可以理解为Activity）中更新UI界面的，比如设置text值、更改大小等，都是不行的。所以我们需要Handler中的消息机制来实现在线程中更新界面。
因为进度条控件在使用的时候都是配合着进度变化不断变更的，通常进度变化都在子线程中来做，当progress值变化时，通过给Handler发送一个消息，携带progress值，Handler在接收消息后处理时将progress更新到UI界面上即可。
Handler的技术原理和具体使用要点可以百度查阅，这里不展开了。下文有使用示例。
## 自定义属性
有了上一段的知识点讲解后，我们可以着实来绘制自定义控件了。绘制之前，我们需要了解下本控件有哪些属性是需要单独定义下，以供用户使用时灵活修改的。初步考虑有：

 - 列表内容
 -  圆环的颜色；
 - 圆环的宽度；
 - 圆环上的进度颜色；
 - 圆环上的进度宽度等。
 
为此我们定义attrs.xml配置文件的内容为：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<resources>
    <!--简单环形进度-->
    <declare-styleable name="SimpleRoundProgress">
        <!--圆环颜色-->
        <attr name="srp_roundColor" format="color" />
        <!--圆环的宽度-->
        <attr name="srp_roundWidth" format="dimension" />
        <!--圆环上的进度颜色-->
        <attr name="srp_progressColor" format="color" />
        <!--圆环上的进度宽度-->
        <attr name="srp_progressWidth" format="dimension" />
        <!--进度值的最大值，一般为100-->
        <attr name="srp_max" format="integer" />
        <!--开始角度，指定进度初始点的绘制位置-->
        <attr name="srp_startAngle" format="integer" />
        <!--样式，空心还是实心-->
        <attr name="srp_style">
            <enum name="STROKE" value="0" />
            <enum name="FILL" value="1" />
        </attr>
    </declare-styleable>
</resources>
```
有了属性定义后，我们在绘制的时候结合这些属性就可以按设置值绘制了。
## 进度条图形绘制
Android自定义图形都需继承View类，所以我们定义的进度条SimpleRoundProgress继承View后，形如：

``` java
/**
 * 简单环形进度条
 */
public class SimpleRoundProgress extends View {
...
}
```
1、读取自定义属性，供后续使用

在SimpleRoundProgress类中定义一些成员变量，用来装载自定义属性的值，一会通过属性加载类来加载属性值。成员变量的定义有：

``` java
public class SimpleRoundProgress extends View {
    private Paint paint; // 画笔对象的引用
    private int roundColor; // 圆环的颜色
    private float roundWidth; // 圆环的宽度
    private int progressColor; // 圆环进度的颜色
    private float progressWidth; // 圆环进度的宽度
    private int max; // 最大进度
    private int style; // 进度的风格，实心或者空心
    private int startAngle; // 进度条起始角度
    public static final int STROKE = 0; // 样式：空心
    public static final int FILL = 1; // 样式：实心
    private int progress; // 当前进度
	...
}
```
在构造函数中读取用户自定义属性被赋予的值。

``` java
public class SimpleRoundProgress extends View {
	...
	public SimpleRoundProgress(Context context) {
        this(context, null);
    }

    public SimpleRoundProgress(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SimpleRoundProgress(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);

        paint = new Paint();

        // 读取自定义属性的值
        TypedArray mTypedArray = context.obtainStyledAttributes(attrs, R.styleable.SimpleRoundProgress);

        // 获取自定义属性和默认值
        roundColor = mTypedArray.getColor(R.styleable.SimpleRoundProgress_srp_roundColor, Color.RED);
        roundWidth = mTypedArray.getDimension(R.styleable.SimpleRoundProgress_srp_roundWidth, 5);
        progressColor = mTypedArray.getColor(R.styleable.SimpleRoundProgress_srp_progressColor, Color.GREEN);
        progressWidth = mTypedArray.getDimension(R.styleable.SimpleRoundProgress_srp_progressWidth, roundWidth);
        max = mTypedArray.getInteger(R.styleable.SimpleRoundProgress_srp_max, 100);
        style = mTypedArray.getInt(R.styleable.SimpleRoundProgress_srp_style, 0);
        startAngle = mTypedArray.getInt(R.styleable.SimpleRoundProgress_srp_startAngle, 90);

        mTypedArray.recycle();
    }
    ...
}
```
2、绘制进度环

绘制进度环主要有两步：绘制背景环、绘制前景进度环。代码均写在View的onDraw方法体内：

``` java
public class SimpleRoundProgress extends View {
	...
    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);
        ...
    }
	...
}
```

1）绘制背景环

因为是圆环型进度条，圆在绘制时圆心位置在整个控件的正中心，所以背景环的绘制代码：

``` java
int centerX = getWidth() / 2; // 获取圆心的x坐标
int radius = (int) (centerX - roundWidth / 2); // 圆环的半径

// step1 画最外层的大圆环
paint.setStrokeWidth(roundWidth); // 设置圆环的宽度
paint.setColor(roundColor); // 设置圆环的颜色
paint.setAntiAlias(true); // 消除锯齿
// 设置画笔样式
switch (style) {
    case STROKE:
        paint.setStyle(Paint.Style.STROKE);
        break;
    case FILL:
        paint.setStyle(Paint.Style.FILL_AND_STROKE);
        break;
}
canvas.drawCircle(centerX, centerX, radius, paint); // 画出圆环
```
作两点说明：

 - 圆心的x/y轴坐标值都是getWidth() / 2（一般宽高一样）；
 - paint.setStyle可以设置画笔是空心型还是实心型，对应的效果就是圆环或圆饼。

2）绘制前景进度环

前景进度环中有涉及到进度的计算，以总进度值为100为例，当进度在50时，如果是横向进度条应该绘制整个控件一半的进度长度，如果是圆环型进度条，应该绘制180度的圆环。依次类推，进度环的角度计算为：

``` java
int sweepAngle = 360 * progress / max; // 计算进度值在圆环所占的角度
```
前景进度环的绘制代码为：

``` java
// step2 画圆弧-画圆环的进度
paint.setStrokeWidth(progressWidth); // 设置画笔的宽度使用进度条的宽度
paint.setColor(progressColor); // 设置进度的颜色
RectF oval = new RectF(centerX - radius , centerX - radius , centerX + radius , centerX + radius ); // 用于定义的圆弧的形状和大小的界限

int sweepAngle = 360 * progress / max; // 计算进度值在圆环所占的角度
// 根据进度画圆弧
switch (style) {
    case STROKE:
        // 空心
        canvas.drawArc(oval, startAngle, sweepAngle, false, paint);
        break;
    case FILL:
        // 实心
        canvas.drawArc(oval, startAngle, sweepAngle, true, paint);
        break;
}
```
至此，进度条界面绘制就完成了，整个SimpleRoundProgress的代码为：

``` java
package com.dommy.loading.widget;

import android.content.Context;
import android.content.res.TypedArray;
import android.graphics.Canvas;
import android.graphics.Color;
import android.graphics.Paint;
import android.graphics.RectF;
import android.util.AttributeSet;
import android.view.View;

import com.dommy.loading.R;

/**
 * 简单环形进度条
 */
public class SimpleRoundProgress extends View {
    private Paint paint; // 画笔对象的引用
    private int roundColor; // 圆环的颜色
    private float roundWidth; // 圆环的宽度
    private int progressColor; // 圆环进度的颜色
    private float progressWidth; // 圆环进度的宽度
    private int max; // 最大进度
    private int style; // 进度的风格，实心或者空心
    private int startAngle; // 进度条起始角度
    public static final int STROKE = 0; // 样式：空心
    public static final int FILL = 1; // 样式：实心
    private int progress; // 当前进度

    public SimpleRoundProgress(Context context) {
        this(context, null);
    }

    public SimpleRoundProgress(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public SimpleRoundProgress(Context context, AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);

        paint = new Paint();

        // 读取自定义属性的值
        TypedArray mTypedArray = context.obtainStyledAttributes(attrs, R.styleable.SimpleRoundProgress);

        // 获取自定义属性和默认值
        roundColor = mTypedArray.getColor(R.styleable.SimpleRoundProgress_srp_roundColor, Color.RED);
        roundWidth = mTypedArray.getDimension(R.styleable.SimpleRoundProgress_srp_roundWidth, 5);
        progressColor = mTypedArray.getColor(R.styleable.SimpleRoundProgress_srp_progressColor, Color.GREEN);
        progressWidth = mTypedArray.getDimension(R.styleable.SimpleRoundProgress_srp_progressWidth, roundWidth);
        max = mTypedArray.getInteger(R.styleable.SimpleRoundProgress_srp_max, 100);
        style = mTypedArray.getInt(R.styleable.SimpleRoundProgress_srp_style, 0);
        startAngle = mTypedArray.getInt(R.styleable.SimpleRoundProgress_srp_startAngle, 90);

        mTypedArray.recycle();
    }

    @Override
    protected void onDraw(Canvas canvas) {
        super.onDraw(canvas);

        int centerX = getWidth() / 2; // 获取圆心的x坐标
        int radius = (int) (centerX - roundWidth / 2); // 圆环的半径

        // step1 画最外层的大圆环
        paint.setStrokeWidth(roundWidth); // 设置圆环的宽度
        paint.setColor(roundColor); // 设置圆环的颜色
        paint.setAntiAlias(true); // 消除锯齿
        // 设置画笔样式
        switch (style) {
            case STROKE:
                paint.setStyle(Paint.Style.STROKE);
                break;
            case FILL:
                paint.setStyle(Paint.Style.FILL_AND_STROKE);
                break;
        }
        canvas.drawCircle(centerX, centerX, radius, paint); // 画出圆环

        // step2 画圆弧-画圆环的进度
        paint.setStrokeWidth(progressWidth); // 设置画笔的宽度使用进度条的宽度
        paint.setColor(progressColor); // 设置进度的颜色
        RectF oval = new RectF(centerX - radius , centerX - radius , centerX + radius , centerX + radius ); // 用于定义的圆弧的形状和大小的界限

        int sweepAngle = 360 * progress / max; // 计算进度值在圆环所占的角度
        // 根据进度画圆弧
        switch (style) {
            case STROKE:
                // 空心
                canvas.drawArc(oval, startAngle, sweepAngle, false, paint);
                break;
            case FILL:
                // 实心
                canvas.drawArc(oval, startAngle, sweepAngle, true, paint);
                break;
        }
    }

    /**
     * 设置进度的最大值
     * <p>根据需要，最大值一般设置为100，也可以设置为1000、10000等</p>
     *
     * @param max int最大值
     */
    public synchronized void setMax(int max) {
        if (max < 0) {
            throw new IllegalArgumentException("max not less than 0");
        }
        this.max = max;
    }

    /**
     * 获取进度
     *
     * @return int 当前进度值
     */
    public synchronized int getProgress() {
        return progress;
    }

    /**
     * 设置进度，此为线程安全控件
     *
     * @param progress 进度值
     */
    public synchronized void setProgress(int progress) {
        if (progress < 0) {
            throw new IllegalArgumentException("progress not less than 0");
        }
        if (progress > max) {
            progress = max;
        }
        this.progress = progress;
        // 刷新界面调用postInvalidate()能在非UI线程刷新
        postInvalidate();
    }
}

```

## 环形进度条的使用
1、页面控件配置

自定义控件在页面中使用时，要使用类名来作为标签，自定义属性要通过命名空间引入，整体如：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    ...
    tools:context="com.dommy.loading.SimpleRoundActivity">

    ...
  
    <com.dommy.loading.widget.SimpleRoundProgress
        android:id="@+id/srp_stroke_0"
        android:layout_width="100dip"
        android:layout_height="100dip"
        android:layout_gravity="center"
        app:srp_max="100"
        app:srp_progressColor="@color/red_web"
        app:srp_roundColor="@color/pro_bg"
        app:srp_roundWidth="6dip"
        app:srp_startAngle="0"
        app:srp_style="STROKE" />

     ...
</LinearLayout>

```
界面编写时的preview效果：
![预览效果](/images/181909/4.png)
因为默认的progress为0，所以没有前景进度颜色。

2、Activity代码控制

通过方法com.dommy.loading.widget.SimpleRoundProgress#setProgress就可以设置控件显示指定的进度，如

``` java
srpStroke0.setProgress(39);
```
的效果为：
![进度值39的效果](/images/181909/5.png)
3、让进度条动起来
当进度值为39时，如果要让进度条动起来，可以让进度值由0-39逐渐递增显示，通过一个线程控制进度值由0-39逐渐递增的代码：

``` java
private void refresh() {
    final int percent = RandomUtil.getRandomPercent();
    new Thread(new Runnable() {
        Message msg = null;

        @Override
        public void run() {
            int start = 0;
            while (start <= percent) {
                msg = new Message();
                msg.what = MSG_REFRESH_PROGRESS;
                msg.arg1 = start;
                handler.sendMessage(msg);
                start++;
                try {
                    Thread.sleep(25);
                } catch (InterruptedException e) {
                }
            }
        }
    }).start();
}
```
这里的percent是一个0-100的随机数，当percent为39时就是我们上述的显示效果。
当线程更新start变量值后，通知handler更新界面状态，handler的定义为：

``` java
private Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handleMessage(msg);
        switch (msg.what) {
            case MSG_REFRESH_PROGRESS:
                srpStroke0.setProgress(msg.arg1);
                break;
        }
    }

};
```
到这里就实现了这样的效果：
![动态变动效果](/images/181909/6.gif)
## 改变效果
由于在图形绘制时读取了自定义属性的值，如果需要更改环形进度的样式，直接通过属性值就可以修改了，比较方便。文章最初贴出来的效果动画中的6个环形进度，就是通过样式更改的出来的，其实质上都是SimpleRoundProgress控件。

如果觉得控件的自定义属性不够用，可以自行添加和使用，在此基础上进行扩展，做出你想要的进度效果即可。
## 源码下载
https://github.com/ahuyangdong/ColorfulLoading
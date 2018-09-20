---
title: Android滑动卡TabLayout+ViewPager布局捕获不到返回键Back事件解决办法
date: 2017-08-28 12:41:41
id: 124141
tags: [TabLayout,双击退出]
categories: Android
---
简介
--

在APP的主页，我们一般都是用Tab卡+ViewPager的方式来构造。这里要说的情况是ViewPager中嵌套的是Activity。（主页个人喜欢用Activity来做ViewPager的视图）

视图布局如下：
![这里写图片描述](https://img-blog.csdn.net/20170828122323769?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

目标
--

我们需要在主页上做事件判断，如果用户连续两次按下返回键，就退出APP，按下一次时给出提醒“再按一次确认退出”。

初步实现
----

在MainActivity中覆写onKeyDown方法，判断返回事件：

``` java
private ViewPager viewPager;
private long exitTime; // 按下返回键的时间

@Override
public boolean onKeyDown(int keyCode, KeyEvent event) {
    if (keyCode == KeyEvent.KEYCODE_BACK && event.getAction() == KeyEvent.ACTION_DOWN) {
        if ((System.currentTimeMillis() - exitTime) > 2000) {
            Snackbar snackbar = Snackbar.make(viewPager, "再按一次退出程序", Snackbar.LENGTH_SHORT);
            snackbar.getView().setBackgroundResource(R.color.colorPrimary);
            snackbar.show();
            exitTime = System.currentTimeMillis();
        } else {
            finish();
        }
        return true;
    }
    return super.onKeyDown(keyCode, event);
}
```
这也是多数人多数情况下的做法。

## 问题 ##
在简介中的布局中应用上述代码时，出现了问题，当APP刚打开时按下返回键直接退出了，onKeyDown并没有被调用，为了排查问题，做了如下验证。

 1. 在ViewPager的每个子视图中添加onKeyDown方法，发现返回键按下时并没有触发onKeyDown，证明子视图也没有捕获到返回事件；
 2. 做几次ViewPager子视图的切换，再按返回键时就有触发onKeyDown，为什么此时又可以捕获到，还没有深究过。
 

解决办法
----
为了解决上述事件捕获不稳定的情况，将上面初步实现的代码替换成以下部分

``` java
private ViewPager viewPager;
private long exitTime; // 按下返回键的时间

@Override
public boolean dispatchKeyEvent(KeyEvent event) {
    if (event.getKeyCode() == KeyEvent.KEYCODE_BACK
            && event.getAction() == KeyEvent.ACTION_DOWN
            && event.getRepeatCount() == 0) {
        // 重写键盘事件分发，onKeyDown方法某些情况下捕获不到，只能在这里写
        if ((System.currentTimeMillis() - exitTime) > 2000) {
            Snackbar snackbar = Snackbar.make(viewPager, "再按一次退出程序", Snackbar.LENGTH_SHORT);
            snackbar.getView().setBackgroundResource(R.color.colorPrimary);
            snackbar.show();
            exitTime = System.currentTimeMillis();
        } else {
            finish();
        }
        return true;
    }
    return super.dispatchKeyEvent(event);
}
```

在此处，我们将onKeyDown的覆写换成了dispatchKeyEvent。

小结
--

根据事件传递顺序，事件分发方法dispatchKeyEvent在onKeyDown之前，初步判断，在上述布局情况下，dispatchKeyEvent捕获到Back事件后，在某些情况下没有继续传递给onKeyDown方法，因此出现捕获不到的情况，在dispatchKeyEvent本身中就可以捕获到。至于为什么没有继续分发，就没有去深究了。
（本文所述问题在sdk26编译的情况下测试出现了，并解决了）
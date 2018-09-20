---
title: Android WebView下拉刷新与SwipeRefreshLayout事件冲突解决
date: 2017-09-01 19:29:40
id: 192940
tags: [SwipeRefreshLayout,WebView]
categories: Android
---
简介
--

本篇介绍WebView下拉刷新方法，另外解决SwipeRefreshLayout与WebView嵌套布局时滑动事件冲突的解决办法。

效果
--
![下拉刷新演示](https://img-blog.csdn.net/20170901174604409?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

下拉刷新
----
SwipeRefreshLayout控件可以优雅的完成下拉事件监听。
1、布局文件：

``` xml
<android.support.v4.widget.SwipeRefreshLayout
    android:id="@+id/swipe_fresh"
    android:layout_width="match_parent"
    android:layout_height="0dip"
    android:layout_weight="1">

    <WebView
        android:id="@+id/webview"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@null"
        android:scrollbars="none" />
        
</android.support.v4.widget.SwipeRefreshLayout>
```

2、Activity

``` java
@BindView(R.id.swipe_fresh)
SwipeRefreshLayout refreshLayout;
@BindView(R.id.webview)
WebView webView;

...
private void initView() {
	refreshLayout.setColorSchemeResources(R.color.colorPrimary);
	refreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
	    @Override
	    public void onRefresh() {
	        webView.loadUrl(url);
	    }
	});
}
...
```
即在SwipeRefreshLayout控件的refresh事件中重新加载WebView的内容就可以了。

事件冲突
----
完成了上述功能后，会发现网页无法上拉，往上滑动就会触发下拉刷新控件的refresh事件。演示：
![事件冲突](https://img-blog.csdn.net/20170901174631281?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

## 解决冲突 ##
关于两个控件的冲突问题，网络上有不少解决办法，有的是自定义SwipeRefreshLayout重写onTouchEvent方法；有的是重写WebView的scroll监听。

最后我找到一个比较好的解决办法。
将Activity代码修改为
``` java
@BindView(R.id.swipe_fresh)
SwipeRefreshLayout refreshLayout;
@BindView(R.id.webview)
WebView webView;

...
private void initView() {
	refreshLayout.setColorSchemeResources(R.color.colorPrimary);
	refreshLayout.setOnRefreshListener(new SwipeRefreshLayout.OnRefreshListener() {
	    @Override
	    public void onRefresh() {
	        webView.loadUrl(url);
	    }
	});
	// 设置子视图是否允许滚动到顶部
	refreshLayout.setOnChildScrollUpCallback(new SwipeRefreshLayout.OnChildScrollUpCallback() {
	   @Override
	   public boolean canChildScrollUp(SwipeRefreshLayout parent, @Nullable View child) {
	       return webView.getScrollY() > 0;
	   }
	});
}
...
```
说明：canChildScrollUp方法返回的true/false表示子视图是否可返回顶部，我们改成webView.getScrollY() > 0后表示webView到顶部时返回false，refreshLayout可接收到下拉动作，触发refresh事件。
此时上拉、下拉都没问题了，看下效果
![最终效果](https://img-blog.csdn.net/20170901174650455?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

源码分析
----

``` java
/**
 * Set a callback to override {@link SwipeRefreshLayout#canChildScrollUp()} method. Non-null
 * callback will return the value provided by the callback and ignore all internal logic.
 * @param callback Callback that should be called when canChildScrollUp() is called.
 */
public void setOnChildScrollUpCallback(@Nullable OnChildScrollUpCallback callback) {
    mChildScrollUpCallback = callback;
}
```
setOnChildScrollUpCallback方法可以设置一定监听接口，返回子视图是否可以滑动到顶部。

``` java
/**
  * Classes that wish to override {@link SwipeRefreshLayout#canChildScrollUp()} method
  * behavior should implement this interface.
  */
public interface OnChildScrollUpCallback {
    /**
    * Callback that will be called when {@link SwipeRefreshLayout#canChildScrollUp()} method
    * is called to allow the implementer to override its behavior.
    *
    * @param parent SwipeRefreshLayout that this callback is overriding.
    * @param child The child view of SwipeRefreshLayout.
    *
    * @return Whether it is possible for the child view of parent layout to scroll up.
    */
    boolean canChildScrollUp(SwipeRefreshLayout parent, @Nullable View child);
}
```
OnChildScrollUpCallback 接口就一个canChildScrollUp方法，返回是否可滚动。

在上述实现过程中，我们判断WebView的状态：

 1. 在顶部时，返回false，表示子视图不可滚动，refreshLayout接收到滑动事件，引出滑动视图和调用滑动刷新方法；
 2. 不在顶部时，webView.getScrollY() > 0，返回true，表示子视图可滚动，refreshLayout中canChildScrollUp()返回true，刷新控件不再处理滑动问题，所以没有调用滑动刷新方法。

参考资料
----

http://blog.csdn.net/u012461368/article/details/51037573
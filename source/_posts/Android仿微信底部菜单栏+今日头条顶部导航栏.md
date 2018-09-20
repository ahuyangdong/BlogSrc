---
title: Android仿微信底部菜单栏+今日头条顶部导航栏
date: 2018-09-12 20:08:27
id: 200827
tags: [微信,今日头条,菜单栏]
categories: Android
---
# 背景
Android应用几乎都会用到底部菜单栏，在Material Design还没有出来之前，TabHost等技术一直占主流，现在Google新sdk中提供了TabLayout类可以便捷的做出底部菜单栏效果。

本节我们实现两种主要的Tab效果：
1. 仿微信底部菜单
2. 仿今日头条顶部导航条

效果预览：
![demo](/images/200827/demo.gif)

# 底部菜单

Tab一般与Activity或Fragment配合使用，以达到多页面切换效果，这里使用Fragment来开发子界面。

微信形式的底部菜单可以理解为一个外层Activity，套用几个Fragment。页面布局层次为：

- MainActivity 主框架
    - MsgFragment 微信
    - ContactFragment 通讯录
    - FindFragment 发现
    - MeFragment 我

做出来的效果：
![weixin](/images/200827/1.png)

1、activity_main.xml布局

ViewPager+TabLayout上下结构：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".MainActivity">

    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="0dip"
        android:layout_weight="1" />

    <View
        android:layout_width="match_parent"
        android:layout_height="0.5dip"
        android:background="@color/line_gray" />

    <android.support.design.widget.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="54dip"
        app:tabIndicatorHeight="0dip" />

</LinearLayout>
```

2、MainActivity源码

``` java
package com.dommy.tab;

import android.os.Bundle;
import android.support.design.widget.Snackbar;
import android.support.design.widget.TabLayout;
import android.support.v4.view.PagerAdapter;
import android.support.v4.view.ViewPager;
import android.support.v7.app.AppCompatActivity;
import android.view.KeyEvent;
import android.view.LayoutInflater;
import android.view.View;
import android.widget.ImageView;
import android.widget.TextView;

import com.dommy.tab.adapter.MainFragmentAdapter;

import butterknife.BindView;
import butterknife.ButterKnife;

/**
 * 主框架
 */
public class MainActivity extends AppCompatActivity {

    /**
     * 菜单标题
     */
    private final int[] TAB_TITLES = new int[]{R.string.menu_msg, R.string.menu_contact, R.string.menu_find, R.string.menu_me};
    /**
     * 菜单图标
     */
    private final int[] TAB_IMGS = new int[]{R.drawable.tab_main_msg_selector, R.drawable.tab_main_contact_selector, R.drawable.tab_main_find_selector
            , R.drawable.tab_main_me_selector};

    @BindView(R.id.view_pager)
    ViewPager viewPager;
    @BindView(R.id.tab_layout)
    TabLayout tabLayout;

    /**
     * 页卡适配器
     */
    private PagerAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ButterKnife.bind(this);

        // 初始化页卡
        initPager();

        setTabs(tabLayout, getLayoutInflater(), TAB_TITLES, TAB_IMGS);
    }

    /**
     * 设置页卡显示效果
     * @param tabLayout
     * @param inflater
     * @param tabTitlees
     * @param tabImgs
     */
    private void setTabs(TabLayout tabLayout, LayoutInflater inflater, int[] tabTitlees, int[] tabImgs) {
        for (int i = 0; i < tabImgs.length; i++) {
            TabLayout.Tab tab = tabLayout.newTab();
            View view = inflater.inflate(R.layout.item_main_menu, null);
            // 使用自定义视图，目的是为了便于修改，也可使用自带的视图
            tab.setCustomView(view);

            TextView tvTitle = (TextView) view.findViewById(R.id.txt_tab);
            tvTitle.setText(tabTitlees[i]);
            ImageView imgTab = (ImageView) view.findViewById(R.id.img_tab);
            imgTab.setImageResource(tabImgs[i]);
            tabLayout.addTab(tab);
        }
    }

    private void initPager() {
        adapter = new MainFragmentAdapter(getSupportFragmentManager());
        viewPager.setAdapter(adapter);

        // 关联切换
        viewPager.addOnPageChangeListener(new TabLayout.TabLayoutOnPageChangeListener(tabLayout));
        tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(TabLayout.Tab tab) {
                // 取消平滑切换
                viewPager.setCurrentItem(tab.getPosition(), false);
            }

            @Override
            public void onTabUnselected(TabLayout.Tab tab) {

            }

            @Override
            public void onTabReselected(TabLayout.Tab tab) {

            }
        });
    }

}

```
源码说明：

- 绑定视图对象使用到了ButterKnife框架；
- viewPager和tabLayout添加了事件互相绑定，这样viewPager的滑动和tab的切换都能相互影响；
- setTabs方法设置tabLayout内部的具体内容，为界面中的TabLayout添加了四个子Tab视图。
- Tab的内容使用到了自定义视图，比较灵活一点，也可以使用Tab自带的布局结构。
 
3、Tab自定义视图item_main_menu.xml
自定义视图包含一个上方图标和下方的文字，使用自定义视图的好处就是图标大小方便修改，文字颜色啥的都好改，比较随心。

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_horizontal"
    android:orientation="vertical">

    <ImageView
        android:id="@+id/img_tab"
        android:layout_width="24dip"
        android:layout_height="24dip"
        android:src="@drawable/menu_msg_default" />

    <TextView
        android:id="@+id/txt_tab"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="2dip"
        android:text="首页"
        android:textColor="@drawable/txt_main_menu_selector"
        android:textSize="11sp" />

</LinearLayout>

```
4、Tab图标selector
以第一个Tab“微信”使用的图标为例，tab_main_msg_selector.xml内容：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:drawable="@drawable/menu_msg_selected" android:state_pressed="true" />
    <item android:drawable="@drawable/menu_msg_selected" android:state_selected="true" />
    <item android:drawable="@drawable/menu_msg_default" />
</selector>
```
使用到了两张图片：menu_msg_default、menu_msg_selected，一张默认图样式，一张选中图样式，对比如下：
![menu_msg_default](/images/200827/5.png)  ![menu_msg_selected](/images/200827/6.png)

5、Tab文字selector
因为Tab选中时需要做区分，所以文字颜色与图标一起变动会更好看，文字颜色也需要写selector。txt_main_menu_selector.xml:

``` xml
<?xml version="1.0" encoding="utf-8"?>
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item android:color="@color/menu_green" android:state_selected="true"></item>
    <item android:color="@color/menu_gray"></item>
</selector>
```
menu_gray是默认状态的灰色，menu_green是选中时呈现的绿色。

6、页面切换Adapter

页面切换内容由viewPager的adapter对象完成，使用Fragment作为子页面时，adapter需要是FragmentPagerAdapter的实例，所以上述代码中的MainFragmentAdapter源码为：

``` java
package com.dommy.tab.adapter;

import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;

import com.dommy.tab.fragment.ContactFragment;
import com.dommy.tab.fragment.FindFragment;
import com.dommy.tab.fragment.MeFragment;
import com.dommy.tab.fragment.MsgFragment;

/**
 * 主界面底部菜单适配器
 */
public class MainFragmentAdapter extends FragmentPagerAdapter {
    public MainFragmentAdapter(FragmentManager fm) {
        super(fm);
    }

    @Override
    public Fragment getItem(int i) {
        Fragment fragment = null;
        switch (i) {
            case 0:
                fragment = new MsgFragment();
                break;
            case 1:
                fragment = new ContactFragment();
                break;
            case 2:
                fragment = new FindFragment();
                break;
            case 3:
                fragment = new MeFragment();
                break;
            default:
                break;
        }
        return fragment;
    }

    @Override
    public int getCount() {
        return 4;
    }

}

```

说明：

- getItem返回具体位置的viewPager切换到i位置时对应的fragment，因为主框架的视图是固定的，所以在这里根据i的值返回对应的fragment对象即可；
- getItem中返回的fragment也可以携带一些参数，如果需要的话；
- getCount返回视图的总数量，这里是固定值4。

7、子页面示例

本例中的子页面只是呈现一个简单的文字，实际开发中根据需要写入相应布局和功能替换即可。这里以MeFragment作为示例：

MeFragment.java：

``` java
package com.dommy.tab.fragment;

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.dommy.tab.R;

/**
 * 我
 */
public class MeFragment extends Fragment {

    public MeFragment() {
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        return inflater.inflate(R.layout.fragment_me, container, false);
    }

}

```
fragment_me.xml：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".fragment.MeFragment">

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="我" />

</RelativeLayout>
```
呈现效果：
![我-界面效果](/images/200827/7.png)

8、小结

MainActivity作为主框架，使用ViewPager实现4个子页面Fragment的切换，使用TabLayout绑定ViewPager来切换视图，实现了Tab卡切换、ViewPager滑动页面的效果，基本实现了微信主框架的效果。

# 顶部导航条
TabLayout放在顶部的时候，加上一些属性配置，就可以完美实现顶部导航的效果。根据导航条样式，这里分为三类：
1. 自适应非固定条数形式；
2. 居中固定条数形式；
3. 平铺固定条数形式。

现就三种效果分别展开，读者可以根据需要选用相应的方法。

## 自适应非固定条数
这种就是和今日头条类似的形式，适用于顶部菜单数量不固定，而且比较多的情况。

从左至右依次排放，每个菜单的内容均完全显示，长度根据内容自动伸缩，超长后的菜单需要滚动显示。

看下效果：
![自适应非固定条数](/images/200827/2.png)

为了方便，源码写在了MsgFragment中，也就是第一个子页面“微信”中。
1、MsgFragment.java
布局结构与MainActivity有类似之处。

``` java
package com.dommy.tab.fragment;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.design.widget.TabLayout;
import android.support.v4.app.Fragment;
import android.support.v4.view.ViewPager;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;

import com.dommy.tab.R;
import com.dommy.tab.adapter.MsgContentFragmentAdapter;

import java.util.ArrayList;
import java.util.List;

import butterknife.BindView;
import butterknife.ButterKnife;

/**
 * 消息
 * <p>在这个界面中实现类似今日头条的头部tab</p>
 */
public class MsgFragment extends Fragment {
    @BindView(R.id.tab_layout)
    TabLayout tabLayout;
    @BindView(R.id.view_pager)
    ViewPager viewPager;

    private MsgContentFragmentAdapter adapter;
    private List<String> names;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        initData();
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_msg, container, false);
        ButterKnife.bind(this, view);

        adapter = new MsgContentFragmentAdapter(getChildFragmentManager());
        viewPager.setAdapter(adapter);
        tabLayout.setupWithViewPager(viewPager);

        // 更新适配器数据
        adapter.setList(names);
        return view;
    }

    private void initData() {
        names = new ArrayList<>();
        names.add("关注");
        names.add("推荐");
        names.add("热点");
        names.add("视频");
        names.add("小说");
        names.add("娱乐");
        names.add("问答");
        names.add("图片");
        names.add("科技");
        names.add("懂车帝");
        names.add("体育");
        names.add("财经");
        names.add("军事");
        names.add("国际");
        names.add("健康");
    }
}

```
2、fragment_msg.xml
布局有主界面比较相似，不过TabLayout被放在顶部了。
``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".fragment.MsgFragment">

    <android.support.design.widget.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="34dip"
        app:tabBackground="@color/white"
        app:tabIndicatorColor="@color/menu_green"
        app:tabIndicatorHeight="1dip"
        app:tabMode="scrollable"
        app:tabMinWidth="40dip"
        app:tabPaddingStart="5dip"
        app:tabPaddingEnd="5dip"
        app:tabSelectedTextColor="@color/wx_head_selected"
        app:tabTextAppearance="@style/tab_head"
        app:tabTextColor="@color/wx_head_default" />

    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="10dip"
        android:layout_weight="1"
        android:background="@color/white" />
</LinearLayout>
```
说明：

> app:tabMode="scrollable"

tabMode的值设为scrollable，表示tab卡过多时自动滑动。

3、MsgContentFragmentAdapter
因为内容页大多近似，所以采用同一个Fragment布局即可，内容根据传参来修改。MsgContentFragmentAdapter.java：

``` java
package com.dommy.tab.adapter;

import android.os.Bundle;
import android.support.v4.app.Fragment;
import android.support.v4.app.FragmentManager;
import android.support.v4.app.FragmentPagerAdapter;

import com.dommy.tab.fragment.MsgContentFragment;

import java.util.ArrayList;
import java.util.List;

/**
 * 消息内容子页面适配器
 */
public class MsgContentFragmentAdapter extends FragmentPagerAdapter {
    private List<String> names;

    public MsgContentFragmentAdapter(FragmentManager fm) {
        super(fm);
        this.names = new ArrayList<>();
    }

    /**
     * 数据列表
     *
     * @param datas
     */
    public void setList(List<String> datas) {
        this.names.clear();
        this.names.addAll(datas);
        notifyDataSetChanged();
    }

    @Override
    public Fragment getItem(int position) {
        MsgContentFragment fragment = new MsgContentFragment();
        Bundle bundle = new Bundle();
        bundle.putString("name", names.get(position));
        fragment.setArguments(bundle);
        return fragment;
    }

    @Override
    public int getCount() {
        return names.size();
    }

    @Override
    public CharSequence getPageTitle(int position) {
        String plateName = names.get(position);
        if (plateName == null) {
            plateName = "";
        } else if (plateName.length() > 15) {
            plateName = plateName.substring(0, 15) + "...";
        }
        return plateName;
    }
}

```
4、MsgContentFragment

子页面只放了一个TextView用来显示参数，MsgContentFragment.java：

``` java
package com.dommy.tab.fragment;

import android.os.Bundle;
import android.support.annotation.Nullable;
import android.support.v4.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import com.dommy.tab.R;

import butterknife.BindView;
import butterknife.ButterKnife;

/**
 * 消息内容页
 */
public class MsgContentFragment extends Fragment {
    @BindView(R.id.txt_content)
    TextView tvContent;

    private String name;

    @Override
    public void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        Bundle bundle = getArguments();
        name = bundle.getString("name");
        if (name == null) {
            name = "参数非法";
        }
    }

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_msg_content, container, false);
        ButterKnife.bind(this, view);

        tvContent.setText(name);
        return view;
    }

}

```

页面布局fragment_msg_content.xml：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".fragment.MsgContentFragment">

    <TextView
        android:id="@+id/txt_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_centerInParent="true"
        android:text="@string/hello_blank_fragment"
        android:textSize="18sp" />

</RelativeLayout>
```



## 居中固定条数
这种形式的导航栏位于水平居中位置，适用于顶部菜单数量较少的情况。

从左至右依次排放，菜单整体位于水平居中位置，因为数量较少，一般不会滚动显示。

看下效果：
![居中固定条数](/images/200827/3.png)
由于界面构建原理与上述内容一致，在此仅说明不同之处：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".fragment.ContactFragment">

    <android.support.design.widget.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="34dip"
        android:background="@color/white"
        app:tabBackground="@color/white"
        app:tabGravity="center"
        app:tabIndicatorHeight="0dip"
        app:tabMinWidth="40dip"
        app:tabMode="fixed"
        app:tabPaddingEnd="5dip"
        app:tabPaddingStart="5dip"
        app:tabSelectedTextColor="@color/wx_head_selected"
        app:tabTextAppearance="@style/tab_head"
        app:tabTextColor="@color/wx_head_default" />

    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="10dip"
        android:layout_weight="1"
        android:background="@color/white" />
</LinearLayout>
```
TabLayout有两个属性与“自适应非固定条数”形式不一样：
> app:tabMode="fixed"

tabMode的值设置为fixed（默认值，也可以不加该属性），表示TabLayout的内容最大长度不会超过自身长度，也就是说不会出现滚动条，添加这个属性时，如果Tab过多，则会比较挤，出现Tab内部内容换行的情况。

> app:tabGravity="center"

tabGravity定义Tab内部的对齐方式，当该属性值为center时，表示居中对齐，不进行拉伸，根据Tab内容自适应宽度。

## 平铺固定条数
Tab平均分配宽度，适用于顶部菜单数量固定，且需要撑满页面的情况。

从左至右依次排放，菜单内容填满整个TabLayout控件，不会滚动显示。

看下效果：
![平铺固定条数](/images/200827/4.png)

在此仅说明不同之处：

``` xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context=".fragment.FindFragment">

    <android.support.design.widget.TabLayout
        android:id="@+id/tab_layout"
        android:layout_width="match_parent"
        android:layout_height="34dip"
        android:background="@color/white"
        app:tabBackground="@color/white"
        app:tabGravity="fill"
        app:tabIndicatorHeight="0dip"
        app:tabMinWidth="40dip"
        app:tabMode="fixed"
        app:tabPaddingEnd="5dip"
        app:tabPaddingStart="5dip"
        app:tabSelectedTextColor="@color/wx_head_selected"
        app:tabTextAppearance="@style/tab_head"
        app:tabTextColor="@color/wx_head_default" />

    <android.support.v4.view.ViewPager
        android:id="@+id/view_pager"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:layout_marginTop="10dip"
        android:layout_weight="1"
        android:background="@color/white" />
</LinearLayout>
```
TabLayout有一个属性与“居中固定条数”形式不一样：

> app:tabGravity="fill"

tabGravity定义Tab内部的对齐方式，当该属性值为fill时，表示填充宽度，会进行拉伸，根据Tab数量平均分配每个Tab的宽度。

# 总结
TabLayout的出现基本解决了以前Android开发遇到的Tab页卡效果不好、不流畅的问题，而且TabLayout还添加了Indicator，能够随手指滑动，修改起来也比较方便。

除了没有直接解决滑动过程中颜色渐变、过渡的问题，普通场景使用TabLayout这个控件已经可以满足需求了。

# 源码
https://github.com/ahuyangdong/TabCustom

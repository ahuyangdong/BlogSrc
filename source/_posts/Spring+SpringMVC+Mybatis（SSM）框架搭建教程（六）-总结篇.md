---
title: Spring+SpringMVC+Mybatis（SSM）框架搭建教程（六）-总结篇
date: 2018-03-14 19:44:27
id: 194427
tags: [SpringMVC,SSM]
categories: Java Web
---
## 背景
前面5篇文章对《Spring+SpringMVC+Mybatis（SSM）框架搭建》进行了分阶段讲解，从Maven插件配置、项目创建、框架配置、实例开发等阶段系统的分析了SSM框架搭建的方法。项目地址：
https://github.com/ahuyangdong/SSMDemo
本篇对前面内容做个总结，并给出SpringMVC框架应用中的常见问题，结合工作内容对web系统常用组件给出选型建议。
## 分篇总结
前5篇文章可总结为：
 1. [Spring+SpringMVC+Mybatis（SSM）框架搭建教程（一）-Maven工具配置](http://www.ahuyangdong.top/2018/02/09/123916) 讲解Maven构建工具的配置方法，铺垫基础；
 2. [Spring+SpringMVC+Mybatis（SSM）框架搭建教程（二）-创建项目](http://www.ahuyangdong.top/2018/02/22/123549) 讲解使用MyEclipse搭建项目的流程，Maven webapp基本结构配置，创建好的项目已经可以运行并看到欢迎页；
 3. [Spring+SpringMVC+Mybatis（SSM）框架搭建教程（三）-配置文件详解](http://www.ahuyangdong.top/2018/02/25/175821) 讲解框架层面的配置设置，包括spring-mvc.xml、spring-mybatis.xml、web.xml；
 4. [Spring+SpringMVC+Mybatis（SSM）框架搭建教程（四）-应用功能开发实例](http://www.ahuyangdong.top/2018/02/28/083107) 讲解使用SSM框架开发小应用的实例，包括两种形式的网络请求处理、Spring注解使用、Controller方法封装；
 5. [Spring+SpringMVC+Mybatis（SSM）框架搭建教程（五）-扩展：过滤器Filter应用](http://www.ahuyangdong.top/2018/03/06/083502) 讲解基于SSM框架的扩展，常用的拦截器配置及应用效果。

## 常见问题
1、Spring中@Resource与@Autowired注解的区别

> @Resource默认按照ByName自动注入，由J2EE提供，需要导入包javax.annotation.Resource。
> @Autowired默认按类型装配（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如@Autowired(required=false) 

我们在实例中基本都用的@Resource，因为我们没有对业务层单独设置name，所以使用@Resource和@Autowired的效果其实没多大区别，就没有去深究差异了。

2、@Repository注解为什么没用？
> @Resource是dao层的注解，在结合Mybatis框架的时候，dao层定义的主要是数据库方法接口层，配置文件spring-mybatis.xml中已明确将dao层纳入扫描，所以在dao层接口上一般不加任何注解，不影响使用。

3、request对象为什么定义成全局的了？不会出现线程并发问题么？
> 众所周知的是，SpringMVC基于方法，所以我们一般不会在注解类（包括Controller）中添加成员变量，因为单例的Controller在多线程环境下，成员变量是会被共享的，所以不安全。
> 在第四篇文章中，我们封装的BaseController中就包含成员变量request对象，这种方法是比较独特的，似乎是SpringMVC框架的特殊处理，在进入每一个Controller方法的时候，request都是相对独立的对象，不会产生共享安全问题。
> 笔者查过资料，验证过上述做法的安全性，但是没有找到可以这样做的真实原因是什么。

4、为什么返回ajax结果是需要带上response参数？
> 因为response不能作为成员变量，否则注入会发生错误。而输出数据又需要PrintWriter通道，所以得在方法上写response参数。

## 常用组件推荐
我们在项目开发过程中尝试过不少web应用组件，后台框架基本都是Spring系列，没什么推荐的。主要是集中在前端部分。
1、web前端框架
前端库推荐[Bootstrap](http://www.bootcss.com/)，非常适合扁平化开发，不过在大型项目中需要做一些个性化扩展才好用。
![Bootstrap](https://img-blog.csdn.net/201802261559492?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
2、弹出层插件
由贤心开发的[layer](http://layer.layui.com/)弹出层，稍稍封装后非常好用，免费美观。
![layer](https://img-blog.csdn.net/20180226155910846?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
3、分页插件
由贤心开发的基于js方式的分页插件[laypage](http://www.layui.com/demo/laypage.html)，由于是纯js渲染的，加载的时候稍有抖动，如果不合适的话可以考虑自己写。像js和jsp版本都可以写一下，这样比较好用。
![laypage](https://img-blog.csdn.net/20180226155747587?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
4、日历插件
用过[laydate](http://www.layui.com/laydate/)，比较美观，但是规则验证不够强大。
![laydate](https://img-blog.csdn.net/20180226155652896?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
用的最舒心的就是经典的[My97DatePicker](http://www.my97.net/)，经久不衰，除了皮肤稍稍过时，其他完全够用，皮肤可以根据需要自定义。
![My97DatePicker](https://img-blog.csdn.net/20180226160036384?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
5、图表统计插件
折线图、柱状图、饼图等，推荐百度的[ECharts](http://echarts.baidu.com/)，美观大方，图库丰富。
![echarts](https://img-blog.csdn.net/20180226155610383?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWh1eWFuZ2Rvbmc=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

## 小结
Spring+SpringMVC+Mybatis（SSM）框架搭建教程从框架jar包配置、请求配置、列表查询、数据请求示例开发等角度都做了讲解，给出了示例代码。
至此，本系列文章已经完结，可以动手搭建一套属于自己的web框架了。
---
title: Java集合工具Map、List、Set、Array使用小结
date: 2017-05-17 17:30:45
id: 173045
tags: [Java,集合工具]
categories: Java
---
应用过程
----
Java代码编写过程中。

目的
--
分析不同场景下不同类、方法的适用性、差异。

场景小结
----------

 1. 需要通过某个id或name取到对应的Object或Entity等，如通过userId取userName。
用Map类，实例化HashMap就可以。

 2. 需要对键值结构参数进行排序或编码，如对http请求参数进行分析。
用Map类，实例化TreeMap，具有按照字典排序功能；如果排序功能不满足需要，可以自定义排序方式。（比较器）

 3. 需要按序显示固定列表，如输出查询结果列表。
用List类，ArrayList。

 4. 需要对列表进行去重，比如计算有多少不重名的文章等。
用Set类，可直接通过List来转换成Set，去除重复数据，不过Set中的重复比较是基于对象的hashCode来做的。

 5. List和Array比较
List存取数据多以get、add方法来操作，而Array则以[角标i]（类似于属性）的方式来操作，个人推荐优先使用List对象。（相比于一些性能上的微小差异，可维护和安全、方便还更重要一些）

 6. List和Map比较
 用途不一样，Map主要是想通过key找到对应的value，和查字典一样；List就是单纯的一个列表，如果想要查找元素，需要通过遍历的方式。

---
title: 推荐一个Laravel-admin 表单字段关联的包
abbrlink: ecf9ade7
date: 2019-12-08 14:09:38
tags: 
	- Laravel
	- Laravel-Admin
---

​		相信有用Laravel 开发web应用的伙伴都有用过Larave-admin。

​		笔者最近遇到了个需求，需要根据表单的某个组件控制其它组件显示或者隐藏。

​				例如：

​						有一个类型radio的组件，选项有官方，和教师。

​						还有一个是教师的select组件。

​						我们需要根据类型radio组件的状态来控制教师的select组件。

![image-20191208143100238](/images/02/Snipaste_2019-12-08_15-07-26.png)

​		那如何实现呢？我翻遍了官方文档，发现并没有提供这一功能。于是我想着要不自己写一个扩展，在写之前我在网上找了各种包，发现了个非常好用的包，嘻嘻~~~

​		最终效果：

![gifhome_544x960_5s](/images/02/gifhome_544x960_5s.gif)

​		



​		向提供者致敬，[传送门](https://github.com/zuweie/FieldInteraction)

​		传送门有详细的使用介绍，这里我就不做教程了，希望对在看的你有帮助。
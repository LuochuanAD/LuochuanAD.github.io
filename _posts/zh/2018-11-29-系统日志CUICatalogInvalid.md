---
layout:     post
title:      CUICatalog Invalid asset name supplied '(null)'
subtitle:   系统日志
date:       2018-11-29
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- error

---

## 问题描述

>工具: Xcode10.X iPhoneX iOS11.x系统 上运行项目, 此时系统打印日志为:

```
[framework] CUICatalog:Invalid asset name supplied:'(null)'

```

## 问题发现与解决


>每当我进入一个新的ViewController时, Xcode都会输出这些系统日志. 后来发现问题出现在我的一个按钮Button 设置的图片是是空的. 如:

```
[UIImage imageNamed:nil];   

```

或者

```
[UIImage imageNamed:@""];   

```


>问题来源:设置button的各个属性 包含图片 都被封装在一个类方法中. …

你可以检查一下你的项目:

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImages1/45.png)


## 结语

* 有的人会在xcode菜单中：Product > Scheme > Edit Scheme > Run
Environment Variables 添加一栏name：OS_ACTIVITY_MODE Value:disable 来屏蔽系统日志.
请不要这样做, 有些项目的bug 会被你忽视掉的.

* 代码规范很重要.

* 封装的意义 不是将简单的东西封装, 而是将复杂,难的东西封装成一个简单的方法或文件等.


博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/84639873](https://blog.csdn.net/luochuanAD/article/details/84639873) 





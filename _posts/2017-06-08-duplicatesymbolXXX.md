---
layout:     post
title:      duplicate symbol _OBJC_CLASS_$_XXX in
subtitle:   系统日志
date:       2017-06-08
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- error

---

## 问题描述

>错误提示:

![](https://img-blog.csdn.net/20170608170033206)

## 问题发现与解决


### 错误种类一:(文件重复,搜索,删掉)

![](https://img-blog.csdn.net/20170608170517168)

### 错误种类二: (导入的是.m文件  (换成.h))

>在CanvasView.m文件里面,系统自动引入了CanvasView.h文件.   


## 结语

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/72927744](https://blog.csdn.net/luochuanAD/article/details/72927744) 





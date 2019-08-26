---
layout:     post
title:      _OBJC_CLASS_$_WKWebView, referenced from...and linker command failed with exit code 1 (use -v...
subtitle:   系统日志
date:       2017-06-07
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- error

---

## 问题描述

>在做JS与OC交互时;Xcode报错:如下图:

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImages1/7.png)

## 问题发现与解决


>(添加Webkit.framework  status:optional)

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImages1/8.png)

Go to your Project, click on General, scroll down to Linked Frameworks and Libraries, and add WebKit.framework as Optional. See here: Xcode 6 + iOS 8 SDK but deploy on iOS 7 (UIWebKit & WKWebKit)

转到你的项目，点击通用，向下滚动到链接的框架和库，添加WebKit。框架是可选的。看这里:Xcode 6+ios8 SDK，但部署在iOS 7上(UIWebKit和WKWebKit)



## 结语

参考:stackoverflow [https://stackoverflow.com/questions/27615041/uiwebview-and-wkwebview](https://stackoverflow.com/questions/27615041/uiwebview-and-wkwebview)

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/72897316](https://blog.csdn.net/luochuanAD/article/details/72897316) 





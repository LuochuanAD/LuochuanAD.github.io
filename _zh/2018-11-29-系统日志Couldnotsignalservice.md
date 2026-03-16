---
layout:     post
title:      Could not signal service com.apple.WebKit.WebContent 113 Could not find specified service
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

>工具: Xcode10.X iPhoneX iOS11.x系统 上运行项目, 每当我进入一个加载网页的ViewController时, 此时系统打印日志为:

```
Could not signal service com.apple.WebKit.WebContent: 113: Could not find specified service

```

## 问题发现与解决


>每当我进入一个加载网页的ViewController时, Xcode都会输出这些系统日志.
问题发现, 当我将wkWebView记载到ViewController.view之前, 就调用了loadRequest:或者loadFileURL: allowingReadAccessToURL:方法, 所以此时系统会输出这些日志.
解决方法 一看就是 先添加wkWebView到 self.view上. 但是, 如果我不解决该问题, 也不会出现崩溃什么的.
这是推荐大家用我封装的LCWebView 可以提高网页的加载速度. 支持iOS7及以上.
GitHub: [https://github.com/LuochuanAD/LCWebView](https://github.com/LuochuanAD/LCWebView)

![](https://raw.githubusercontent.com/LuochuanAD/LCWebView/master/LCWebViewDemo/LCWebView/demo.gif)


## 结语

* 有的人会在xcode菜单中：Product > Scheme > Edit Scheme > Run
Environment Variables 添加一栏name：OS_ACTIVITY_MODE Value:disable 来屏蔽系统日志.
请不要这样做, 有些项目的bug 会被你忽视掉的.

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/84640163](https://blog.csdn.net/luochuanAD/article/details/84640163) 





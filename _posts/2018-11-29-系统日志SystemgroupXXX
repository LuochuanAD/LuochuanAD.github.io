---
layout:     post
title:      [MC] System group container for systemgroup.com.apple.configurationprofiles path is /private...
subtitle:   [MC]
date:       2018-11-29
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- 系统日志

---

## 问题描述

>工具: Xcode10.X iPhoneX iOS11.x系统 上运行项目 当我使用 NavigationBar push到一个新的ViewController时. 会有一段短暂的黑色方块. push动画结束后, 往返几次. 不再出现该情况. 此时系统打印日志为:

```
[MC] System group container for systemgroup.com.apple.configurationprofiles path is /private/var/containers/Shared/SystemGroup/systemgroup.com.apple.configurationprofiles

[MC] Reading from public effective user settings.

```

## 问题发现与解决


### 一,情况1:

>当我push到一个新的ViewController时,我的键盘是跟着push动画从右往左同步出现. 那么问题发现了. 键盘调起方法是发生在viewDidLoaded方法之前. 在需要在要跳转的ViewController的 viewDidloaded方法中 写调用键盘的代码.
当我无视该系统日志时, 程序也不会崩溃. 同时在iOS12.x的系统上, apple已经修复该问题.

### 二,情况2:

>隐私权限问题. 需要在在info.plist文件配置相应设置,如

```
<key>NSPhotoLibraryUsageDescription</key>
<string>此 App 需要您的同意才能读取媒体资料库</string>

```


## 结语

有的人会在xcode菜单中：Product > Scheme > Edit Scheme > Run
Environment Variables 添加一栏name：OS_ACTIVITY_MODE Value:disable 来屏蔽系统日志.
请不要这样做, 有些项目的bug 会被你忽视掉的.

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/84639464](https://blog.csdn.net/luochuanAD/article/details/84639464) 





---
layout:     post
title:     Xcode15之安装iOS 17 Simulator Runtime
subtitle:   安装iOS17
date:       2023-9-19
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Xcode15 
- iOS17

---

# 前言

>2023/09/19 Xcode15正式发布了,同时iOS17也正式登场.

## 先在AppStore下载Xcode15后,出现了以下问题

### 问题一:iOS 17无法通过Xcode15安装.

报错信息:Failed with HTTP status 400: bad request Domain: DataGatheringNSURLSessionDelegate Code: 1 User Info: { DVTErrorCreationDateKey = "2023-09-20 01:12:47 +0000"; }

### 解决方案:

1, 在苹果网站下载iOS_17_Simulator_Runtime.dmg安装包(安装包地址:https://download.developer.apple.com/Developer_Tools/iOS_17_Simulator_Runtime/iOS_17_Simulator_Runtime.dmg )

2, 在终端输入:
```
xcrun simctl runtime add /xxx...你的绝对路径...xxx/iOS_17_Simulator_Runtime.dmg

```

参考链接: https://stackoverflow.com/questions/77133646/ios-17-0-simulator-21a328-failed-with-http-status-400-bad-request


### 问题二:Xcode15编译项目报错

Error 'DT_TOOLCHAIN_DIR cannot be used to evaluate LIBRARY_SEARCH_PATHS, use TOOLCHAIN_DIR instead

### 解决方案:

>听说cocoapods会在1.3.0版本会解决此问题. 但是我等不了了,目前cocoapod版本为1.2.1, 解决方法如下:

在Podfile文件中添加以下代码. 注意:请不要直接替换,先查看原有的Podfile文件中的代码. 我这里(M1芯片)原本设置了(config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"),现在将两者合并为新的配置代码.
```
post_install do |installer|
      installer.pods_project.targets.each do |target|
          target.build_configurations.each do |config|
	  config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64" ///此处为原PodFile的配置代码,不需要可删除这一行
          xcconfig_path = config.base_configuration_reference.real_path
          xcconfig = File.read(xcconfig_path)
          xcconfig_mod = xcconfig.gsub(/DT_TOOLCHAIN_DIR/, "TOOLCHAIN_DIR")
          File.open(xcconfig_path, "w") { |file| file << xcconfig_mod }
          end
      end
  end
```


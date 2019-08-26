---
layout:     post
title:      自定义SFSafariViewController
subtitle:   SFSafariViewController
date:       2019-08-25
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- SFSafariViewController

---

## 前言

>iOS9.0新出了一个加载网页的控件:SFSafariViewControoler, 它具有Safari浏览器的众多优点,加载更快,兼容更强. 但也有人反映SFSafariViewController的自定义不强, 接下来看看它详细的API讲解以及Demo样例.

自定义的样式为下图:

![](https://raw.githubusercontent.com/LuochuanAD/LCSafariViewControllerDemo/master/LCSafariViewControllerDemo/example.png)


## 分析

>关键词：SFSafariViewController

### 一,导入

>导入#import <SafariServices/SafariServices.h> 并且遵守SFSafariViewControllerDelegate代理


>详细代码LCSafariViewController已上传到[Github](https://github.com/LuochuanAD/LCSafariViewControllerDemo) 



### 二,SFSafariViewController初始化 (两种方法)

```
SFSafariViewController * sf;

/*
*  初始化方法一:适用iOS9.0及以上
*
*  @param url 注意:1,不能加载本地HTML文件(已在模拟器测试过); 2,URL需要加https://或者http:// 否则会崩溃
*/
sf = [[SFSafariViewController alloc]initWithURL:[NSURL URLWithString:@"https://strictfrog.com"]];//API_AVAILABLE(ios(9.0, *))


/*
*  初始化方法二:适用iOS9.0~iOS11.0 和 iOS11.0及以上
*
*  @param url 注意:1,不能加载本地HTML文件(已在模拟器测试过); 2,URL需要加https://或者http:// 否则会崩溃
*
*  @param entersReaderIfAvailable 是否能开启阅读模式 只适用iOS9.0~iOS11.0
*
*  @param SFSafariViewControllerConfiguration 配置文件 适用iOS11.0及以上 属性entersReaderIfAvailable:是否能开启阅读模式; 属性barCollapsingEnabled: 滑动网页时是否隐藏导航栏和标签栏
*/
if (@available(iOS 11.0, *)) {
SFSafariViewControllerConfiguration * sfConfig = [[SFSafariViewControllerConfiguration alloc]init];//API_AVAILABLE(ios(11.0))
sfConfig.entersReaderIfAvailable = YES;
sfConfig.barCollapsingEnabled = YES;
sf = [[SFSafariViewController alloc]initWithURL:[NSURL URLWithString:@"https://strictfrog.com"] configuration:sfConfig];
}else{
sf = [[SFSafariViewController alloc]initWithURL:[NSURL URLWithString:@"https://strictfrog.com"] entersReaderIfAvailable:YES];//API_AVAILABLE(ios(9.0, 11.0))
}

sf.delegate = self;

```

### 三,SFSafariViewController的属性设置


```

/*
*  设置SFSafariViewController属性
*
*  preferredBarTintColor:导航栏和标签栏背景颜色 (建议run一下) 只适用iOS10.0及以上
*
*  preferredControlTintColor 导航栏和标签栏里的字颜色 (建议run一下) 只适用iOS10.0及以上
*
*  dismissButtonStyle 返回按钮的样式 Done/Cancel/Close 这三种字都已做了文字国际化 只适用iOS11.0及以上
*/
if (@available(iOS 10.0, *)) {
sf.preferredBarTintColor = [UIColor lightGrayColor];
sf.preferredControlTintColor = [UIColor blackColor];

if (@available(iOS 11.0, *)) {
sf.dismissButtonStyle = SFSafariViewControllerDismissButtonStyleCancel;
}
}

```


### 四,跳转方式(2种方法)


```
/*
*  跳转方式一: 默认
*/
[self presentViewController:sf animated:YES completion:nil];


/*
*  自定义跳转方式二:
*/
[self.navigationController pushViewController:sf animated:YES];


```
>如果想使用push跳转, 需要做额外一些代码设置

```

- (void)viewDidAppear:(BOOL)animated{
[super viewDidAppear:animated];
self.navigationController.navigationBar.hidden = NO;

}

- (void)viewWillDisappear:(BOOL)animated{
[super viewWillDisappear:animated];
//自定义跳转方式二 设置
self.navigationController.navigationBar.hidden = YES;

}


//此方法为SFSafariViewControllerDelegate代理方法
- (void)safariViewControllerDidFinish:(SFSafariViewController *)controller{
//自定义跳转方式二 设置
[self.navigationController popViewControllerAnimated:YES];
}

```

### 五,SFSafariViewControllerDelegate代理方法


```
/*
*  点击左上角Done/Cancel/Close按钮
*/
- (void)safariViewControllerDidFinish:(SFSafariViewController *)controller{
//自定义跳转方式二 设置
[self.navigationController popViewControllerAnimated:YES];
}


```



```
/*
*  只有初始化加载页面完成加载一次
*
*  @param didLoadSuccessfully  YES:加载成功;NO:加载失败
*/
- (void)safariViewController:(SFSafariViewController *)controller didCompleteInitialLoad:(BOOL)didLoadSuccessfully{

}

```


```

/*
*  重定向url 只适用iOS11.0及以上
*
*  @param URL  网页重定向时会调用该方法,并且返回重定向页面的URL
*/
- (void)safariViewController:(SFSafariViewController *)controller initialLoadDidRedirectToURL:(NSURL *)URL API_AVAILABLE(ios(11.0)){

}
```



```
/*
*  过滤safari浏览器点击分享按钮,弹出窗口中的内容
*
*  @param URL 当前网页的URL
*
*  @param title 当前网页的title
*
*  @result 返回一个数组, 数组中包含你要过滤掉的UIActivityType类型, 默认是不过滤
*/
- (NSArray<UIActivityType> *)safariViewController:(SFSafariViewController *)controller excludedActivityTypesForURL:(NSURL *)URL title:(nullable NSString *)title API_AVAILABLE(ios(11.0)){
/**
UIActivityTypePostToFacebook      发送到Facebook
UIActivityTypePostToTwitter       发送到Twitter
UIActivityTypePostToWeibo         发送到微博
UIActivityTypeMessage             发送到Message
UIActivityTypeMail                发送到Mail
UIActivityTypePrint               打印
UIActivityTypeCopyToPasteboard    拷贝到粘贴板
UIActivityTypeAssignToContact     发送给指定联系人
UIActivityTypeSaveToCameraRoll    保存到相机
UIActivityTypeAddToReadingList    添加到阅读标签
UIActivityTypePostToFlickr        发送到Flicker
UIActivityTypePostToVimeo         发送到Vimeo
UIActivityTypePostToTencentWeibo  发送到腾讯微博
UIActivityTypeAirDrop             传送给Airdrop
UIActivityTypeOpenInIBooks        打开iBooks
UIActivityTypeMarkupAsPDF         作为PDF收藏
*/
return @[UIActivityTypeAddToReadingList];
}
```

>最后一个代理方法,如果有需求,就需要一个自定义的UIActivity,详细看Demo



```

/*
*  点击分享按钮,UIActivityViewController弹出窗口将要显示的时候, 调用该方法
*
*  @param URL 当前网页的URL
*
*  @param title 当前网页的title
*
*  @result 返回一个UIActivity数组, 可以自定义添加UIActivity到数组中
*/

- (NSArray<UIActivity *> *)safariViewController:(SFSafariViewController *)controller activityItemsForURL:(NSURL *)URL title:(nullable NSString *)title{
LCActivity * activity = [[LCActivity alloc]init];


return @[activity];
}

```

### 六,自定义UIActivity

>覆盖UIActivity的几个方法就好


```

#import <UIKit/UIKit.h>

@interface LCActivity : UIActivity
@property (nonatomic, copy) void(^activityBlock)(void);

@end

#import "LCActivity.h"

@implementation LCActivity
- (NSString *)activityType {

return @"CustomActivity";
}

- (NSString *)activityTitle {

return @"自定义Activity";
}

- (UIImage *)activityImage {//只有iOS8才支持彩色,所以不建议使用彩色照片


return [UIImage imageNamed:@"luochuan"];
}

- (void)performActivity {
[self activityDidFinish:YES];

if (self.activityBlock) {
self.activityBlock();
}
}

- (BOOL)canPerformWithActivityItems:(NSArray *)activityItems {

if (activityItems.count > 0) {

return YES;
}

return NO;
}

@end


```


## 结语

SFSafariViewController所有的属性及方法 全都讲解完毕. 如果有不理解的可以对照github上代码查看







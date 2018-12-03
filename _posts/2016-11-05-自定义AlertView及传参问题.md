---
layout:     post
title:      iOS之自定义AlertView的传参问题
subtitle:   自定义AlertView
date:       2016-11-08
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- AlertView

---

## 前言

>最近在开发SDK的过程中,有一个需求是:点击按钮,弹框显示姓名和身份证号,同时在弹框中可以修改姓名和身份证号.  
分析:需要自定义一个AlertView 和双向传值(需要用到Block).
自定义的样式为下图:

![](https://raw.githubusercontent.com/LuochuanAD/OC-CommonAlertView/master/LCAlertView/demoExample2016110501.png)


## 分析

>关键词：自定义View

### 一,思路 

>NewFile 继承自UIView. 添加2个Block : cancelBlock 和 sureBlock 分别用于 自定义AlertView的2个按钮点击事件,之后就是UI页面的搭建了

```
typedef void(^cancelBlock)();
typedef void(^sureBlock)(NSString*name,NSString*idnumber);

@interface CommonAlertView : UIView

@property(nonatomic, strong)cancelBlock cancel_block;
@property(nonatomic, strong)sureBlock sure_block;

@end
```
>详细代码CommonAlertView已上传到[Github](https://github.com/LuochuanAD/OC-CommonAlertView) 



### 二,如何使用CommonAlertView 

```
CommonAlertView *alertView=[CommonAlertView alertViewWithCancelbtnClicked:^{

} andSureBtnClicked:^(NSString *name, NSString *idnumber) {
//保存修改后的姓名和身份证号
oriName=name;
oriIdnumber=idnumber;

} withName:oriName withidcard:oriIdnumber];
UIView *keywindow=[UIApplication sharedApplication].keyWindow;
[keywindow addSubview:alertView];

```

### 三,代码分析

>在CommonAlertView.h中定义了弹框的返回和修改按钮的Block用来处理按钮的点击事件. 
点击修改按钮需要将姓名和身份证号的值传出去,所以修改按钮的block会有2个参数(注:根据你项目的需求,修改参数的多少). 
接下来,就是创建自定义View的方法,我用类方法,带有4个参数:2个是弹框的返回和修改按钮事件的block,和要初始显示在弹框的姓名和身份证号参数.

```
+(instancetype)alertViewWithCancelbtnClicked:(cancelBlock) cancelBlock 
andSureBtnClicked:(sureBlock) sureBlock 
withName:(NSString *)name 
withidcard:(NSString *)idnumber;

```
>在CommonAlertView.m
中的initWithFrame:方法中中创建遮罩blackView和弹框alertView,

```

- (instancetype)initWithFrame:(CGRect)frame{

self=[super initWithFrame:frame];
if (self) {
...............
[self addSubview:_alertview];
[self animationAlert:self.alertview];
}
return self;
}

```

>同时给alertview的加上弹出的动画(注:这里的动画可自定义,方法为animationAlert:). 

```
//弹框动画自定义(可以修改参数改变动画)
- (void)animationAlert:(UIView *)view{
CAKeyframeAnimation *popAnimation=[CAKeyframeAnimation animationWithKeyPath:@"transform"];
popAnimation.duration=0.6;
popAnimation.values=@[[NSValue valueWithCATransform3D:CATransform3DMakeScale(0.01f, 0.01f, 1.0f)],[NSValue valueWithCATransform3D:CATransform3DMakeScale(1.1f, 1.1f, 1.0f)],[NSValue valueWithCATransform3D:CATransform3DMakeScale(0.9f, 0.9f, 1.0f)],[NSValue valueWithCATransform3D:CATransform3DIdentity]];
popAnimation.keyTimes=@[@0.0f,@0.5f,@0.75f];
popAnimation.timingFunctions=@[[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut],[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut],[CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut]];
[view.layer addAnimation:popAnimation forKey:nil];

}
```

>在layoutSubviews的方法中添加控件到弹框alertView上并布局.

```
- (void)layoutSubviews{
[super layoutSubviews];
_nameLable=[[UILabel alloc]initWithFrame:CGRectMake(10, 20, 68, 30)];
...............

}
```

>在+(instancetype)alertViewWithCancelbtnClicked:(cancelBlock) cancelBlock andSureBtnClicked:(sureBlock) sureBlock withName:(NSString *)name withidcard:(NSString *)idnumber中定义2个NSString属性分别接收name和idnumber的值,并对定义的2个属性重写,否则会无法显示. 

```
+(instancetype)alertViewWithCancelbtnClicked:(cancelBlock) cancelBlock andSureBtnClicked:(sureBlock) sureBlock withName:(NSString *)name withidcard:(NSString *)idnumber{
CommonAlertView *alertView=[[CommonAlertView alloc]initWithFrame:CGRectMake(0, 0, LCWINDOWS.width,LCWINDOWS.height)];
alertView.center=CGPointMake(LCWINDOWS.width/2, LCWINDOWS.height/2-64);//view的中点
alertView.cancel_block=cancelBlock;//取消block
alertView.sure_block=sureBlock;//确定block
alertView.name=[NSString stringWithFormat:@"%@",name];//将传入的姓名参数接收
alertView.idnumber=[NSString stringWithFormat:@"%@",idnumber];//将传入的身份证号参数接收
return alertView;

}
//name字符串属性重写(注意:NSString类型的属性都要重写,不然会出现无法显示到界面的情况)
- (void)setName:(NSString *)name{

_name=[NSString stringWithFormat:@"%@",name];
}
//idnumber字符串属性重写
- (void)setIdnumber:(NSString *)idnumber{

_idnumber=[NSString stringWithFormat:@"%@",idnumber];
}
```
>在返回和修改按钮的点击事件中将block块传出去.在修改按钮的事件中要将姓名和身份证号的输入框中的值带入到sure_block中.

```
//取消按钮点击
- (void)cancelBtnClicked{
[UIView animateWithDuration:0.3 animations:^{
self.alpha=0;
} completion:^(BOOL finished) {
[self removeFromSuperview];
self.alertview=nil;
self.cancel_block();
}];


}
//确定按钮点击
- (void)sureBtnClicked{
//如果输入的姓名和身份证号不符合正则表达式,则return;(这里不做验证,请根据项目需求修改)
[UIView animateWithDuration:0.3 animations:^{
self.alpha=0;
} completion:^(BOOL finished) {
[self removeFromSuperview];
self.alertview=nil;
//将新输入的姓名和身份证号值传出去.
self.sure_block(_nameTextFild.text,_idcardTextFild.text);
}];

}

```

## 结语

这个自定义alertView很简单. 同时我的语言组织的不好,请原谅,这是我2016年来第一次写的博客.见笑了.

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/53081461](https://blog.csdn.net/luochuanAD/article/details/53081461) 





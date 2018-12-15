---
layout:     post
title:      iOS之UsefulCategory
subtitle:   分类category
date:       2018-04-13
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- category


---


***

***一,Usefulcategory说明***

***
部分category收集于blog,实际项目等等中. 本人对其重新"筛选,整理,优化,封装".每个分类都有详细的解释和使用说明.很有用!
**Github地址:  [https://github.com/LuochuanAD/UsefulCategory](https://github.com/LuochuanAD/UsefulCategory "https://github.com/LuochuanAD/UsefulCategory")**
***
***二,Usefulcategory使用***

***

**- NSObject(Safe)**
![NSObject(Safe)](https://img-blog.csdn.net/20180413151039594?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
导入到项目中, 此分类需在MRC环境下使用,需添加-fno-objc-arc.  防止崩溃,提高代码的健壮性.强烈推荐(因为我自己写的没有它好,这个分类是用的别人).  该分类包含NSString,NSMutableString,NSArray,NSMutableArray,NSDictionary,NSMutableDictionary,NSAttributedString,NSMutableAttributedString,NSSet,NSMutableSet,NSOrderedSet,NSMutableOrderedSet,NSUserDefaults,NSCache.

**- UITableView**
![UITableView](https://img-blog.csdn.net/20180413152258571?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```python
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSUInteger,CustomAnimation){
CustomAnimationTypeMove,        //左侧飞入
CustomAnimationTypeAlpha,       //透明
CustomAnimationTypeFall,        //上面掉落
CustomAnimationTypeShaKe,       //抖动动画
CustomAnimationTypeOverTurn,    //翻转动画
CustomAnimationTypeToTop,       //从下往上
CustomAnimationTypeSpringList,  //从上往下弹动动画
CustomAnimationTypeShrinkToTop, //从下往上挤向顶部
CustomAnimationTypeLayDown,     //从上往下展开
CustomAnimationTypeRote,        //翻转动画
};

@interface UITableView (CellAnimation)
/**
*  UItableView 显示动画
*
*  @param type 动画类型
*/
- (void) customAnimation:(CustomAnimation)type;
```

**- NSData**
![NSData](https://img-blog.csdn.net/20180413152523382?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#import <Foundation/Foundation.h>

typedef NS_ENUM(NSUInteger,DataEncryption){
DataEncryptionMD2,
DataEncryptionMD4,
DataEncryptionMD5,
DataEncryptionSHA1,
DataEncryptionSHA224,
DataEncryptionSHA256,
DataEncryptionSHA384,
DataEncryptionSHA512,
};

@interface NSData (Encryption)

/*
*  NSData 加密
*
*  @param type 加密类型
*/
- (NSData *) dataEncryptionType:(DataEncryption)type;
```
**- NSObject**
![NSObject](https://img-blog.csdn.net/20180413154009912?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
*NSObject+Category.h*
```python
#import <Foundation/Foundation.h>
#import <objc/runtime.h>

@interface NSObject (Category)
@property (nonatomic, strong, readonly) NSMutableArray *associatedObjectNames;

/**
*  为当前object动态增加分类
*
*  @param propertyName   分类名称
*  @param value  分类值
*  @param policy 分类内存管理类型
*/
- (void)objc_setAssociatedObject:(NSString *)propertyName value:(id)value policy:(objc_AssociationPolicy)policy;
/**
*  获取当前object某个动态增加的分类
*
*  @param propertyName 分类名称
*
*  @return 值
*/
- (id)objc_getAssociatedObject:(NSString *)propertyName;
/**
*  删除动态增加的所有分类
*/
- (void)objc_removeAssociatedObjects;

/**
*  获取对象的所有属性
*
*  @return 属性dict
*/
- (NSArray *)getProperties;
```
*NSObject+ModelWithDictionary.h*
```python
#import <Foundation/Foundation.h>

@interface NSObject (ModelWithDictionary)

/**
*  Model转字典
*/
- (NSDictionary*)toDictionary;
/**
*  字典转Model (警告:字典只能为一层结构)
*/
-(id) initWithDictionary:(NSDictionary*)dictionary;
```
*NSObject+PerformBlock.h*
```python
#import <Foundation/Foundation.h>

@interface NSObject (PerformBlock)
// try catch
+ (NSException *)tryCatch:(void(^)())block;
+ (NSException *)tryCatch:(void(^)())block finally:(void(^)())aFinisheBlock;

/**
*  在主线程运行block
*
*  @param aInMainBlock 被运行的block
*/
+ (void)performInMainThreadBlock:(void(^)())aInMainBlock;
/**
*  延时在主线程运行block
*
*  @param aInMainBlock 被运行的block
*  @param delay        延时时间
*/
+ (void)performInMainThreadBlock:(void(^)())aInMainBlock afterSecond:(NSTimeInterval)delay;
/**
*  在非主线程运行block
*
*  @param aInThreadBlock 被运行的block
*/
+ (void)performInThreadBlock:(void(^)())aInThreadBlock;
/**
*  延时在非主线程运行block
*
*  @param aInThreadBlock 被运行的block
*  @param delay          延时时间
*/
+ (void)performInThreadBlock:(void(^)())aInThreadBlock afterSecond:(NSTimeInterval)delay;
```
*NSObject+helper.h*
```python
#import <Foundation/Foundation.h>

@interface NSObject (helper)

//交换类方法
+ (void)swizzleClassMethod:(SEL)origSelector withMethod:(SEL)newSelector;

//交换实例方法
- (void)swizzleInstanceMethod:(SEL)origSelector withMethod:(SEL)newSelector;

@end
```
**- NSString**
![NSString](https://img-blog.csdn.net/20180413154522458?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
*NSString+Category.h*
```python
#import <Foundation/Foundation.h>
#import <UIKit/UIKit.h>

@interface NSString (Category)

/**
*  计算文字高度
*
*  @param fontSize 字号
*
*  @param fixedWidth 控件宽度
*/
- (CGFloat)countHeightOfTextWithFontSize:(CGFloat)fontSize fixedWidth:(CGFloat)fixedWidth;

/**
*  计算文字宽度
*
*  @param fontSize 字号
*
*  @param fixedHeight 控件高度
*/
- (CGFloat)countWidthOfTextWithFontSize:(CGFloat)fontSize fixedHeight:(CGFloat)fixedHeight;
```
*NSString+Predicate.h*
```python
#import <Foundation/Foundation.h>

typedef NS_ENUM(NSUInteger, PredicateType){
PredicateType_MobileNo,          //检测有效的电话号码
PredicateType_RealName,          //检测有效的真实姓名
PredicateType_Chinese,           //检测是否只有中文
PredicateType_VerificationCode,  //检测有效的验证码(根据自家的验证码位数进行修改)
PredicateType_BankCardNumber,    //检测有效的银行卡号
PredicateType_Email,             //检测有效的邮箱
PredicateType_LettersAndNumbers, //检测有效的字母和数字
PredicateType_IDnumberFor15,     //检测有效身份证 15位
PredicateType_IDnumberFor18,     //检测有效身份证 18位
};

@interface NSString (Predicate)
/**
*  验证字符串 是否符合正则表达式
*
*  @param type 正则表达式类型
*/
- (BOOL)isValidStringWithPredicateType:(PredicateType)type;
```
*NSString+TimeStamp.h*
```python
#import <Foundation/Foundation.h>

@interface NSString (TimeStamp)

/**
*  通过时间戳计算时间差（几小时前、几天前
*
*  @param compareDate 需要对比的时间戳
*/
+ (NSString *) compareCurrentTime:(NSTimeInterval) compareDate;

/**
*  通过时间戳得出对应的时间
*
*  @param timestamp 时间戳
*/
+ (NSString *) getDateStringWithTimestamp:(NSTimeInterval)timestamp;

/**
*  通过时间戳和时间格式 显示时间
*
*  @param timestamp 时间戳
*
*  @param formatter 格式
*/
+ (NSString *) getStringWithTimestamp:(NSTimeInterval)timestamp formatter:(NSString*)formatter;

```
**- UIButton**
![UIButton](https://img-blog.csdn.net/20180413154950464?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#import <UIKit/UIKit.h>
@interface UIButton (EnlargeTouchArea)

/**
*  扩大或缩小 UIButton 的点击范围
*
*  @param top    向上增加的点击范围(注:top参数值为负数,则反方向减小点击范围)
*
*  @param right  向右增加的点击范围(注:right参数值为负数,则反方向减小点击范围)
*
*  @param bottom 向下增加的点击范围(注:bottom参数值为负数,则反方向减小点击范围)
*
*  @param left   向左增加的点击范围(注:left参数值为负数,则反方向减小点击范围)
*/
- (void)setEnlargeEdgeWithTop:(CGFloat)top right:(CGFloat)right bottom:(CGFloat)bottom left:(CGFloat)left;

@end
```
**- UIColor**
![UIColor](https://img-blog.csdn.net/20180413155310328?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#import <UIKit/UIKit.h>
/**
*  功能:通过RGB创建颜色
*
*  @param red <CGFloat> <范围:0~255.0>
*  @param green <CGFloat> <范围:0~255.0>
*  @param blue <CGFloat> <范围:0~255.0>
*
*  @return UIColor
*
*  example: rgb(173.0,23.0,11.0)
*/
UIColor *rgb(CGFloat red, CGFloat green, CGFloat blue);

/**
*  功能:通过RGB以及alpha创建颜色
*
*  @param red <CGFloat> <范围:0~255.0>
*  @param green <CGFloat> <范围:0~255.0>
*  @param blue <CGFloat> <范围:0~255.0>
*  @param alpha <CGFloat> <范围:0~1.0>
*
*  @return UIColor
*
*  example: rgbA(173.0,23.0,11.0,0.5)
*/
UIColor *rgbA(CGFloat red, CGFloat green, CGFloat blue, CGFloat alpha);


@interface UIColor (Category)

/**
*  Create a color from a HEX string.
*  It supports the following type:
*  - #RGB
*  - #ARGB
*  - #RRGGBB
*  - #AARRGGBB
*
*  @param hexString NSString
*
*  @return Returns the UIColor instance
*/
+ (UIColor *)hex:(NSString *)hexString;

/**
*  通过0xffffff的16进制数字创建颜色
*
*  @param aRGB 0xffffff
*
*  @return UIColor
*/
+ (UIColor *)colorWithRGB:(NSUInteger)aRGB;
@end

```
**- UIImage**
![UIImage](https://img-blog.csdn.net/20180413155447282?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSUInteger,ImageBlurType) {
ImageBlurSoft,
ImageBlurLight,
ImageBlurExtraLight,
ImageBlurDark,
};
@interface UIImage (Category)

/**
*  创建由颜色生成的图片
*
*  @param color 生成图片的色值
*
*  @param rect  图片的frame设置
*
*  @param roundSize 图片的圆角大小
*
*  @param corners 圆角的类型(左上,右上,左下,右下)
*
*  @param text 图片上的文字
*/
+ (UIImage *)creatImageWithCustomColor:(UIColor *)color rect:(CGRect)rect andRoundSize:(CGFloat)roundSize byRoundingCorners:(UIRectCorner)corners text:(NSString*)text;


/**
*  设置图片的圆角及边框
*
*  @param radius 圆角大小
*
*  @param borderWidth 边框宽度
*
*  @param borderColor 边框颜色
*/
- (UIImage *)setImageWithRoundCornerRadius:(CGFloat)radius
borderWidth:(CGFloat)borderWidth
borderColor:(UIColor *)borderColor;

/**
*  图片旋转角度
*
*  @param degrees 角度
*/
- (UIImage *) imageRotatedByDegrees:(CGFloat)degrees;

/**
*  图片旋转 (效果自己Run)
*
*  @param horizontal 横向
*
*  @param vertical 竖向
*/
- (UIImage *)flipHorizontal:(BOOL)horizontal vertical:(BOOL)vertical;

/**
*  玻璃化效果，这里与系统的玻璃化枚举效果一样，但只是一张图
*
*  @param type 玻璃化效果类型
*/
- (UIImage *)createImageWithBlur:(ImageBlurType)type;

/**
*  玻璃化效果，这里与系统的玻璃化枚举效果一样，但只是一张图
*
*  @param tintColor 自定义玻璃化效果颜色
*/
- (UIImage *)imageByBlurWithTint:(UIColor *)tintColor;
/**
*  自定义玻璃化效果，这里与系统的玻璃化枚举效果一样，但只是一张图;(自定义各种参数)
*/
- (UIImage *)imageByBlurRadius:(CGFloat)blurRadius
tintColor:(UIColor *)tintColor
tintMode:(CGBlendMode)tintBlendMode
saturation:(CGFloat)saturation
maskImage:(UIImage *)maskImage;

/**
*  设置图片的模糊度
*
*  @param blur 模糊度0~1之间
*
*  @param exclusionPath 模糊路径(可不做设置)
*/
- (UIImage *) boxblurImageWithBlur:(CGFloat)blur exclusionPath:(UIBezierPath *)exclusionPath;

@end
```
**- UILable**
![UILable](https://img-blog.csdn.net/20180413155604706?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#import <UIKit/UIKit.h>

@interface UILabel (AutoResize)
/**
*  UIlable控件高度自适应文字实际计算的高度 (注:lable.numberOfLines=0)
*
*  @param text lable.text
*/
- (void)setAutoResizeWithText:(NSString*)text;

@end
```
**- UITabBar**
![UITabBar](https://img-blog.csdn.net/20180413155658394?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#import <UIKit/UIKit.h>

typedef NS_ENUM(NSUInteger, CustomBadgeType){
kCustomBadgeStyleRedDot,
kCustomBadgeStyleNumber,
kCustomBadgeStyleNone
};

@interface UITabBar (CustomBadge)

/**
*  设置tab上icon的宽度，用于调整badge的位置
*/
- (void)setTabIconWidth:(CGFloat)width;

/**
*  设置badge的top
*/
- (void)setBadgeTop:(CGFloat)top;

/**
*  @param type 设置badge类型
*
*  @param badgeValue 数值
*
*  @param index   第几个tabbarItem (从0开始算)
*/
- (void)setBadgeStyle:(CustomBadgeType)type value:(NSInteger)badgeValue atIndex:(NSInteger)index;

@end
```
***
***三,UsefulHeader.h***

***
![UsefulHeader](https://img-blog.csdn.net/20180413155911494?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2x1b2NodWFuQUQ=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
```python
#ifndef UsefulHeader_h
#define UsefulHeader_h

/*
*  WINDOWS 屏幕宽高 (包括横竖屏)
*/
#if __IPHONE_OS_VERSION_MAX_ALLOWED >= 80000
#define WINDOWS_WIDTH ([[UIScreen mainScreen] respondsToSelector:@selector(nativeBounds)]?[UIScreen mainScreen].nativeBounds.size.width/[UIScreen mainScreen].nativeScale:[UIScreen mainScreen].bounds.size.width)
#define WINDOWS_HEIGHT ([[UIScreen mainScreen] respondsToSelector:@selector(nativeBounds)]?[UIScreen mainScreen].nativeBounds.size.height/[UIScreen mainScreen].nativeScale:[UIScreen mainScreen].bounds.size.height)
#define WINDOWS ([[UIScreen mainScreen] respondsToSelector:@selector(nativeBounds)]?CGSizeMake([UIScreen mainScreen].nativeBounds.size.width/[UIScreen mainScreen].nativeScale,[UIScreen mainScreen].nativeBounds.size.height/[UIScreen mainScreen].nativeScale):[UIScreen mainScreen].bounds.size)
#else
#define WINDOWS_WIDTH [UIScreen mainScreen].bounds.size.width
#define WINDOWS_HEIGHT [UIScreen mainScreen].bounds.size.height
#define WINDOWS [UIScreen mainScreen].bounds.size
#endif

/*
*  设备类型
*/
#define iPhone4_4s ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(640, 960), [[UIScreen mainScreen] currentMode].size) : NO)
#define iPhone5_5s_5c_se ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(640, 1136), [[UIScreen mainScreen] currentMode].size) : NO)
#define iPhone6_6s_7_8 ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(750, 1334), [[UIScreen mainScreen] currentMode].size) : NO)
#define iPhone6p_6sp_7p_8p ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1242, 2208), [[UIScreen mainScreen] currentMode].size) : NO)
#define iPhoneX ([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1125, 2436), [[UIScreen mainScreen] currentMode].size) : NO)
#define is_iPhoneXSeries ((int)((SCREEN_HEIGHTL/SCREEN_WIDTHL)*100) == 216)?YES:NO

#define ISIPAD ([[UIDevice currentDevice] userInterfaceIdiom]==UIUserInterfaceIdiomPad)
#define ISIPHONE   ([[UIDevice currentDevice] userInterfaceIdiom]==UIUserInterfaceIdiomPhone)
/*
*  TABBARHEIGHT tabbar高度;NAVIGATIONBARHEIGHT 导航栏高度
*/
#define TABBARHEIGHT (is_iPhoneXSeries?83:49)
#define NAVIGATIONBARHEIGHT (is_iPhoneXSeries?88:64)

/*
*  ColorHex 16进制颜色(example:ColorHex(0xe5e5e5))
*  RGBA rgba颜色
*/
#define ColorHex(hexValue) [UIColor colorWithRed:((float)((hexValue & 0xFF0000) >> 16))/255.0 green:((float)((hexValue & 0xFF00) >> 8))/255.0 blue:((float)(hexValue & 0xFF))/255.0 alpha:1]
#define RGBA(r,g,b,a) [UIColor colorWithRed:r/255.0 green:g/255.0 blue:b/255.0 alpha:a]
/*
*  SLog  NSLog封装宏
*/
#define SLog(format, ...) printf("class: <%p %s:(%d) > method: %s \n%s\n", self, [[[NSString stringWithUTF8String:__FILE__] lastPathComponent] UTF8String], __LINE__, __PRETTY_FUNCTION__, [[NSString stringWithFormat:(format), ##__VA_ARGS__] UTF8String] )

/*
*  AppCurrentLanguage APP当前语言
*/
#define AppCurrentLanguage ([[NSLocale preferredLanguages] objectAtIndex:0])

/*
*  WeakSelf 弱引用;StrongSelf 强引用
*/
#define WeakSelf        @weakify(self);
#define StrongSelf      @strongify(self);
#ifndef weakify
#if DEBUG
#if __has_feature(objc_arc)
#define weakify(object) autoreleasepool{} __weak __typeof__(object) weak##_##object = object;
#else
#define weakify(object) autoreleasepool{} __block __typeof__(object) block##_##object = object;
#endif
#else
#if __has_feature(objc_arc)
#define weakify(object) try{} @finally{} {} __weak __typeof__(object) weak##_##object = object;
#else
#define weakify(object) try{} @finally{} {} __block __typeof__(object) block##_##object = object;
#endif
#endif
#endif

#ifndef strongify
#if DEBUG
#if __has_feature(objc_arc)
#define strongify(object) autoreleasepool{} __typeof__(object) strong##_##object = weak##_##object;
#else
#define strongify(object) autoreleasepool{} __typeof__(object) block##_##object = block##_##object;
#endif
#else
#if __has_feature(objc_arc)
#define strongify(object) try{} @finally{} __typeof__(object) strong##_##object = weak##_##object;
#else
#define strongify(object) try{} @finally{} __typeof__(object) block##_##object = block##_##object;
#endif
#endif
#endif

/*
*  GCD_ONCE_BLOCK 单例;
*/
#define GCD_ONCE_BLOCK(onceBlock) static dispatch_once_t onceToken; dispatch_once(&onceToken, onceBlock);
/*
*  GCD_MAIN_THREAD 主线程;
*/
#define GCD_MAIN_THREAD(mainQueueBlock) dispatch_async(dispatch_get_main_queue(), mainQueueBlock);
/*
*  GCD_GLOBAL_QUEUE_DEFAULT 子线程;
*/
#define GCD_GLOBAL_QUEUE_DEFAULT(globalQueueBlock) dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), globalQueueBlocl);
/*
*  GCD_MAIN_DELAY 不堵塞线程并在主线程的延迟执行
*/
#define GCD_MAIN_DELAY(timer,block) dispatch_after(dispatch_time(DISPATCH_TIME_NOW, INT64_C(timer) * NSEC_PER_SEC), dispatch_get_main_queue(), block)
/*
*  StringIsEmpty 字符串是否为空
*/
#define StringIsEmpty(str) ([str isKindOfClass:[NSNull class]] || str == nil || [str length] < 1 ? YES : NO )
/*
*  ArrayIsEmpty 数组是否为空
*/
#define ArrayIsEmpty(array) (array == nil || [array isKindOfClass:[NSNull class]] || array.count == 0)
/*
*  DictIsEmpty 字典是否为空
*/
#define DictIsEmpty(dic) (dic == nil || [dic isKindOfClass:[NSNull class]] || dic.allKeys == 0)
/*
*  ObjectIsEmpty 对象是否为空
*/
#define ObjectIsEmpty(_object) (_object == nil \
|| [_object isKindOfClass:[NSNull class]] \
|| ([_object respondsToSelector:@selector(length)] && [(NSData *)_object length] == 0) \
|| ([_object respondsToSelector:@selector(count)] && [(NSArray *)_object count] == 0))
/*
*  FONT 字体大小
*/
#define FONT(F) [UIFont fontWithName:@"FZHTJW--GB1-0" size:F]

#endif /* UsefulHeader_h */

```


## 结语


博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/79930273](https://blog.csdn.net/luochuanAD/article/details/79930273) 





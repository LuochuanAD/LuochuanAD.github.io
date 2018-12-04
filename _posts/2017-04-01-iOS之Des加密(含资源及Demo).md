---
layout:     post
title:      iOS之Des加密(含资源及Demo)
subtitle:   DESEncryption
date:       2017-04-01
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- DES加密

---

## 前言

>最近有人在群中问我des加密的事,告诉我网上的将des的资料很少. 会者不难,难着不会. 资源:[GithubDemo](https://github.com/LuochuanAD/DESEncryption)

## 分析

>在做des加密,解密时,你需要和服务端约定2个字段.  

第一个字段:比如"xxxx"  这个字段和做图片上传约定的字段一个道理, 便于服务器唯一识别;

第二个字段:为iV[ ]  这个字段会在下图,及Demo中标注出来.


## 使用DESEncryption 

```
#import <CommonCrypto/CommonDigest.h>

#import <CommonCrypto/CommonCryptor.h>

#import "GTMBase64.h"

```
>由于GTMBase64是MRC,所以:要添加 -fno-objc-arc
![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxuz85p0qij31qs0tu459.jpg)

### 三,代码分析

>在CommonAlertView.h中定义了弹框的返回和修改按钮的Block用来处理按钮的点击事件. 
点击修改按钮需要将姓名和身份证号的值传出去,所以修改按钮的block会有2个参数(注:根据你项目的需求,修改参数的多少). 
接下来,就是创建自定义View的方法,我用类方法,带有4个参数:2个是弹框的返回和修改按钮事件的block,和要初始显示在弹框的姓名和身份证号参数.

```
/*字符串加密
 *参数
 *plainText : 加密明文
 *key        : 密钥 64位 为和后台约定的字段示例为xxxx
 */
+ (NSString *) encryptUseDES:(NSString *)plainText key:(NSString *)key
{
    NSString *ciphertext = nil;
    const char *textBytes = [plainText UTF8String];
    NSUInteger dataLength = [plainText length];
    unsigned char buffer[1024];
    memset(buffer, 0, sizeof(char));
    Byte iv[] = { 0x12, 0x34, 0x56, 0x78,  0x90,  0xAB,  0xCD,  0xEF };
    //注:iv[] = { 0x12, 0x34, 0x56, 0x78,  0x90,  0xAB,  0xCD,  0xEF }为移动端和后台约定的字段
    size_t numBytesEncrypted = 0;
    CCCryptorStatus cryptStatus = CCCrypt(kCCEncrypt, kCCAlgorithmDES,
                                          kCCOptionPKCS7Padding,
                                          [key UTF8String], kCCKeySizeDES,
                                          iv,
                                          textBytes, dataLength,
                                          buffer, 1024,
                                          &numBytesEncrypted);
    if (cryptStatus == kCCSuccess) {
        NSData *data = [NSData dataWithBytes:buffer length:(NSUInteger)numBytesEncrypted];
        
        ciphertext = [[NSString alloc] initWithData:[GTMBase64 encodeData:data] encoding:NSUTF8StringEncoding];
    }
    return ciphertext;
}
//解密
+ (NSString *) decryptUseDES:(NSString*)cipherText key:(NSString*)key
{
    NSData* cipherData = [GTMBase64 decodeString:cipherText];
    unsigned char buffer[1024];
    memset(buffer, 0, sizeof(char));
    size_t numBytesDecrypted = 0;
    Byte iv[] = { 0x12, 0x34, 0x56, 0x78,  0x90,  0xAB,  0xCD,  0xEF };
    //注:iv[] = { 0x12, 0x34, 0x56, 0x78,  0x90,  0xAB,  0xCD,  0xEF }为移动端和后台约定的字段
    CCCryptorStatus cryptStatus = CCCrypt(kCCDecrypt,
                                          kCCAlgorithmDES,
                                          kCCOptionPKCS7Padding,
                                          [key UTF8String],
                                          kCCKeySizeDES,
                                          iv,
                                          [cipherData bytes],
                                          [cipherData length],
                                          buffer,
                                          1024,
                                          &numBytesDecrypted);
    NSString* plainText = nil;
    if (cryptStatus == kCCSuccess) {
        NSData* data = [NSData dataWithBytes:buffer length:(NSUInteger)numBytesDecrypted];
        plainText = [[NSString alloc] initWithData:data encoding:NSUTF8StringEncoding];
    }
    return plainText;
}



```

## DES加密常见的问题

* 我曾将手机号DES加密成字符串,上传到服务器,发现有一些手机号加密后的字段有加号("+") 服务端会将加号转为空格  ,导致解密不出来. 那么解决方法是:给des加密后的字符串进行url编码, 再上传服务器.
* Des加密数组是出现过丢失的情况,请看这一篇文章:[iOS之POST请求数组样式参数DES加密问题](https://strictfrog.com/2016/11/16/iOS%E4%B9%8BPOST%E8%AF%B7%E6%B1%82%E6%95%B0%E7%BB%84%E6%A0%B7%E5%BC%8F%E5%8F%82%E6%95%B0DES%E5%8A%A0%E5%AF%86%E9%97%AE%E9%A2%98/)

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanad/article/details/68945526](https://blog.csdn.net/luochuanad/article/details/68945526) 





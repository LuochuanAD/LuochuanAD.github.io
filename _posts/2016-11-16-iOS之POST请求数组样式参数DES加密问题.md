---
layout:     post
title:      iOS之POST请求数组样式参数DES加密问题
subtitle:   url编码
date:       2016-11-16
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- DES
- URL编码

---

## 前言

>最近使用POST请求时会出现参数丢失情况

## 分析
 
>在用post请求时,字典中的一个参数为数组形式,下图为json的格式:

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxuzhmb6ztj30sq0dywg1.jpg)


```
//错代码
NSMutableDictionary *dict=[[NSMutableDictionary alloc]init];
    [dict setObject:self.array forKey:@"dataArray"];
    [dict setObject:@"1" forKey:@"type"];
    [dict setObject:@"xxx" forKey:@"test"];
    NSDictionary *postDic=[NSDictionary dictionaryWithDictionary:dict];


```
>我为什么说是错误代码:因为post上传的参数中数组是不能用小括号表示,要用中括号表示,(具体,可用控制台打印,如果数组是用小括号输出,则是不符合上传服务器格式的) 正确的格式如下图: 

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxuzj8dj1fj30o20awab8.jpg)

>正确的代码是 将数组转为json字符串,代码如下:

```
//正确代码
NSMutableDictionary *dict=[[NSMutableDictionary alloc]init];
    NSData *data=[NSJSONSerialization dataWithJSONObject:self.array options:NSJSONWritingPrettyPrinted error:nil];
    NSString *jsonStr=[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    [dict setObject:jsonStr forKey:@"dataArray"];
    [dict setObject:@"1" forKey:@"type"];
    [dict setObject:@"xxx" forKey:@"test"];
    NSDictionary *postDic=[NSDictionary dictionaryWithDictionary:dict];



```


>接下来有个新的需求,需要将dataArray对应的value数组des加密.(关于des加密,网上有非常多的讲解,这里只讲des加密遇到的问题)
分析,和后台约定好iV和密匙(注:正确分析), 直接将数组转为的json字符串加密就可以了(注:错误分析).,以下为错误代码:

```
//错误代码
NSMutableDictionary *dict=[[NSMutableDictionary alloc]init];
    NSData *data=[NSJSONSerialization dataWithJSONObject:self.array options:NSJSONWritingPrettyPrinted error:nil];
    NSString *jsonStr=[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    NSString *jsonDES=[LCdes lcEncryUseDES:jsonStr];//此方法为des加密方法
    [dict setObject:jsonDES forKey:@"dataArray"];
    [dict setObject:@"1" forKey:@"type"];
    [dict setObject:@"xxx" forKey:@"test"];
    NSDictionary *postDic=[NSDictionary dictionaryWithDictionary:dict];


```
>如果用以上代码加密, 然后在本地进行解密输出解密后的数组,如下图:

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxuzl4ecw4j30xa0ds409.jpg)

>有没有发现什么? 是不是少了}]大括号和中括号.  这时有人问,是不是字符串太长,导致加密时丢了一部分,所以解密后,少了}].   答案:不是字符串过长导致.   (注:字符串过长导致加密解密丢失,我会在文章最后给出解决办法.)  现在我在数组中在增加几组数据, 本地解密后输出如下图:

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxuzmg8513j30pa0g6abt.jpg)

>这回发现了吧,某个解密后的数据都少了大括号和中括号.  原因我也找了很久,说是由于des加密的填充方式导致加密的字符串末端是不固定的.  好吧,说说解决办法办:就是先将json字符串进行url编码,然后再加密. 这样就不会出现丢掉大括号和中括号的情况. 一定要先编码后加密,如果顺序错了,仍然是解决不了的. 以下是正确代码:


```

//正确代码
NSMutableDictionary *dict=[[NSMutableDictionary alloc]init];
    NSData *data=[NSJSONSerialization dataWithJSONObject:self.array options:NSJSONWritingPrettyPrinted error:nil];
    NSString *jsonStr=[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding];
    NSString *jsonDES=[LCdes lcEncryUseDES:[LCdes UrlValueEncode:jsonStr]];//先url编码,后des加密
    [dict setObject:jsonDES forKey:@"dataArray"];
    [dict setObject:@"1" forKey:@"type"];
    [dict setObject:@"xxx" forKey:@"test"];
    NSDictionary *postDic=[NSDictionary dictionaryWithDictionary:dict];


```

>控制台输出:

 ![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxuznwa485j31sg088wij.jpg)
 
>将数组解密,再解密得到:正确的格式

![](https://ws1.sinaimg.cn/large/006tNbRwgy1fxuzosjjj7j30o20awab8.jpg)

>看到这里,接下来说一下真正的由于字符串太长,导致des加密解密丢失的解决办法:
在des加密和解密的方法中将长度增加:如下图:

![](https://ws3.sinaimg.cn/large/006tNbRwgy1fxuzpos2a3j30u010y4da.jpg)


## 结语

关于DES加密的讲解及Demo,请看这篇文章:[iOSDES(Des资源及Demo)](https://strictfrog.com/2017/04/01/iOS%E4%B9%8BDes%E5%8A%A0%E5%AF%86(%E5%90%AB%E8%B5%84%E6%BA%90%E5%8F%8ADemo)/)

>总结就一句话, 将数组转为json字符串,再将json字符串先url编码,然后des加密.     后台获取到该参数时,先des解密,在url解码.  
下面附上URL编码和解码的代码:

```
//url编码
+ (NSString *)UrlValueEncode:(NSString *)str{
    NSString *result = (NSString *)CFBridgingRelease(CFURLCreateStringByAddingPercentEscapes(kCFAllocatorDefault,
                                                                           (CFStringRef)str,
                                                                           NULL,
                                                                           CFSTR("!*'();:@&=+$,/?%#[]"),
                                                                           kCFStringEncodingUTF8));
    
    return result;
}
//url解码
+ (NSString *)decodeFromPercentEscapeString: (NSString *) input
{
    NSMutableString *outputStr = [NSMutableString stringWithString:input];
    [outputStr replaceOccurrencesOfString:@"+"
                               withString:@" "
                                  options:NSLiteralSearch
                                    range:NSMakeRange(0, [outputStr length])];
    
    return [outputStr stringByReplacingPercentEscapesUsingEncoding:NSUTF8StringEncoding];
}



```

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/53188544](https://blog.csdn.net/luochuanAD/article/details/53188544) 





---
layout:     post
title:      iOS之支持https与ssl双向验证
subtitle:   自定义AlertView
date:       2016-11-30
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- SSL

---

## 前言

>在WWDC 2016开发者大会上，苹果宣布了一个最后期限：到2017年1月1日 App Store中的所有应用都必须启用 App Transport Security安全功能。App Transport Security（ATS）是苹果在iOS 9中引入的一项隐私保护功能，屏蔽明文HTTP资源加载，连接必须经过更安全的HTTPS。苹果目前允许开发者暂时关闭ATS，可以继续使用HTTP连接，但到年底所有官方商店的应用都必须强制性使用ATS。

![](https://ws2.sinaimg.cn/large/006tNbRwgy1fxv03xaiv9j30to0jwn0r.jpg)


## 分析

>项目说明: 一开始听到这个消息,我对ATS的概念一知半解. 经过5天的时间,终于搞好了,在此记录一下.  我的项目中用到AFNetwork3.x,ASIHttpRequest,UIWebView.   在文章后面会一一讲解. 
我们后台开发的同学,用了nginx服务器,直接将http转为HTTPS.开发只用了一周.
首先后台会给你提供很多的证书,但是客户端只要几个证书,下面根据你的需求,选择证书.

![](https://ws4.sinaimg.cn/large/006tNbRwgy1fxv05ilxhoj30me0se0uz.jpg) 

>现在项目使用AFN3.x  所有接口支持HTTPS.对此更新. :
证书;只要一个client.p12证书及对应的密码如:lovely_girl
现在以一个AFN3.x请求为例,代码如下

```
- (void)startRequest:(NSDictionary*)params path:(NSString*)path{
    
   urlString=[NSStringstringWithFormat:@"%@%@",Httpsource,path];
   _manager=[AFHTTPSessionManagermanager];
   _manager.responseSerializer=[AFHTTPResponseSerializerserializer];
   _manager.requestSerializer.timeoutInterval=12;
   _manager.securityPolicy.allowInvalidCertificates=YES;
    [_manager.securityPolicysetValidatesDomainName:NO];
    [_manager.requestSerializersetValue:@"gzip"forHTTPHeaderField:@"Accept-Encoding"];
    
   __weak typeof(self) weakSelf =self;
    [_managersetSessionDidReceiveAuthenticationChallengeBlock:^NSURLSessionAuthChallengeDisposition(NSURLSession *session,NSURLAuthenticationChallenge *challenge,NSURLCredential *__autoreleasing *_credential) {
       
       NSURLSessionAuthChallengeDisposition disposition =NSURLSessionAuthChallengePerformDefaultHandling;
       __autoreleasing NSURLCredential *credential =nil;
       if ([challenge.protectionSpace.authenticationMethodisEqualToString:NSURLAuthenticationMethodServerTrust]) {
           if ([weakSelf.manager.securityPolicyevaluateServerTrust:challenge.protectionSpace.serverTrustforDomain:challenge.protectionSpace.host]) {
                credential = [NSURLCredentialcredentialForTrust:challenge.protectionSpace.serverTrust];
               if (credential) {
                    disposition =NSURLSessionAuthChallengeUseCredential;
                }else {
                    disposition =NSURLSessionAuthChallengePerformDefaultHandling;
                }
            }else {
                disposition =NSURLSessionAuthChallengeCancelAuthenticationChallenge;
            }
        }else {
           SecIdentityRef identity = NULL;
           SecTrustRef trust = NULL;
           NSString *p12 = [[NSBundlemainBundle] pathForResource:@"client"ofType:@"p12"];
           NSFileManager *fileManager =[NSFileManagerdefaultManager];
            
           if(![fileManager fileExistsAtPath:p12])
            {
               NSLog(@"client.p12:not exist");
            }
           else
            {
               NSData *PKCS12Data = [NSDatadataWithContentsOfFile:p12];
               if ([HttpRequestextractIdentity:&identity andTrust:&trustfromPKCS12Data:PKCS12Data])
                {
                   SecCertificateRef certificate = NULL;
                   SecIdentityCopyCertificate(identity, &certificate);
                   const void*certs[] = {certificate};
                   CFArrayRef certArray =CFArrayCreate(kCFAllocatorDefault, certs,1,NULL);
                    credential =[NSURLCredentialcredentialWithIdentity:identity certificates:(__bridge NSArray*)certArray persistence:NSURLCredentialPersistencePermanent];
                    disposition =NSURLSessionAuthChallengeUseCredential;
                }
            }
        }
        *_credential=credential;
       return disposition;
    }];
 
    [_managerPOST:urlStringparameters:params progress:^(NSProgress *_Nonnull uploadProgress) {
    }success:^(NSURLSessionDataTask *_Nonnull task, id _Nullable responseObject) {
       NSDictionary *dic=[NSJSONSerializationJSONObjectWithData:responseObject options:NSJSONReadingAllowFragmentserror:nil];
       if ([_delegaterespondsToSelector:@selector(request:didReceiveData:)]) {
            [_delegaterequest:selfdidReceiveData:dic];
        }
    }failure:^(NSURLSessionDataTask *_Nullable task, NSError *_Nonnull error) {
       if ([_delegaterespondsToSelector:@selector(requestFailed:)]) {
            [_delegaterequestFailed:self];
        }
    }];
   if ([_delegaterespondsToSelector:@selector(requestStarted:)]) {
        [_delegaterequestStarted:self];
    }
}
 
+(BOOL)extractIdentity:(SecIdentityRef *)outIdentity andTrust:(SecTrustRef*)outTrust fromPKCS12Data:(NSData *)inPKCS12Data
{
    OSStatus securityError =errSecSuccess;
    NSDictionary *optionsDictionary = [NSDictionarydictionaryWithObject:@"lovely_girl"forKey:(id)kSecImportExportPassphrase];
    
    CFArrayRef items =CFArrayCreate(NULL,0, 0,NULL);
    securityError = SecPKCS12Import((CFDataRef)inPKCS12Data,(CFDictionaryRef)optionsDictionary,&items);
    
    if (securityError ==0) {
        CFDictionaryRef myIdentityAndTrust =CFArrayGetValueAtIndex (items, 0);
        constvoid *tempIdentity = NULL;
        tempIdentity = CFDictionaryGetValue (myIdentityAndTrust,kSecImportItemIdentity);
        *outIdentity = (SecIdentityRef)tempIdentity;
        constvoid *tempTrust = NULL;
        tempTrust = CFDictionaryGetValue (myIdentityAndTrust,kSecImportItemTrust);
        *outTrust = (SecTrustRef)tempTrust;
    } else {
        NSLog(@"--------证书错误------- %d",(int)securityError);
        returnNO;
    }
    returnYES;
}


```
只要写这2个方法,就可以完整的以一个自签证书,进行https接口请求.

>完整的加载一个自签的https的网页,新建HTTPSURLProtocol继承NSURLProtocol.


1,在appdelegate.m中注册,添加以下代码:


```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
        [NSURLProtocol registerClass:[HTTPSURLProtocol class]];
}


```

2,HTTPSURLProtocol.h

```
#import <Foundation/Foundation.h>
 
/**
	该类用于处理https类型的ajax请求
 */
@interface HTTPSURLProtocol : NSURLProtocol
{
    NSMutableData* _data;
    NSURLConnection *theConncetion;
}
 
@end


```

3,HTTPSURLProtocol.m

```
#import "HTTPSURLProtocol.h"
 
static NSString * const URLProtocolHandledKey = @"这里任意字符串";
@implementation HTTPSURLProtocol
 
+ (BOOL)canInitWithRequest:(NSURLRequest *)theRequest
{
    //FIXME: 后来的https类型的reqeust应该也要被处理
    NSString *scheme = [[theRequest URL] scheme];
    if ( ([scheme caseInsensitiveCompare:@"http"] == NSOrderedSame ||
          [scheme caseInsensitiveCompare:@"https"] == NSOrderedSame))
    {
        //看看是否已经处理过了，防止无限循环
        if ([NSURLProtocol propertyForKey:URLProtocolHandledKey inRequest:theRequest]) {
            return NO;
        }
//此处做个判断,app.xxxx.com是我们项目的域名.任何不属于我们域名的不做处理,这样可以加载https://weibo.com等第三方https网页
        if (![[[theRequest URL] host] isEqualToString:@"app.xxxx.com"]) {
            return NO;
        }
        return YES;
    }
    return NO;
}
 
+ (NSURLRequest*)canonicalRequestForRequest:(NSURLRequest*)request
{
    NSMutableURLRequest *mutableReqeust = [request mutableCopy];
    mutableReqeust = [self redirectHostInRequset:mutableReqeust];
    return mutableReqeust;
}
 
- (void)startLoading
{
    
    
    //theConncetion = [[NSURLConnection alloc] initWithRequest:self.request delegate:self];
    
    NSMutableURLRequest *mutableReqeust = [[self request] mutableCopy];
    //标示改request已经处理过了，防止无限循环
    [NSURLProtocol setProperty:@YES forKey:URLProtocolHandledKey inRequest:mutableReqeust];
    theConncetion = [NSURLConnection connectionWithRequest:mutableReqeust delegate:self];
    if (theConncetion) {
        _data = [NSMutableData data];
    }
}
 
- (void)stopLoading
{
    // NOTE:如有清理工作，可以在此处添加
    [theConncetion cancel];
}
 
+ (BOOL)requestIsCacheEquivalent:(NSURLRequest*)requestA toRequest:(NSURLRequest*)requestB
{
    return [super requestIsCacheEquivalent:requestA toRequest:requestB];
}
 
#pragma mark - NSURLConnectionDelegate
 
- (BOOL)connection:(NSURLConnection *)connection canAuthenticateAgainstProtectionSpace:(NSURLProtectionSpace *)protectionSpace
{
    //响应服务器证书认证请求和客户端证书认证请求
    return [protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust] ||
    [protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodClientCertificate];
}
 
- (void)connection:(NSURLConnection *)connection didReceiveAuthenticationChallenge:(NSURLAuthenticationChallenge *)challenge
{
    NSURLCredential* credential;
    if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodServerTrust])
    {
        //服务器证书认证
        credential= [NSURLCredential credentialForTrust:challenge.protectionSpace.serverTrust];
    }
    else if ([challenge.protectionSpace.authenticationMethod isEqualToString:NSURLAuthenticationMethodClientCertificate])
    {
        //客户端证书认证
        //TODO:设置客户端证书认证
       
        NSString *thePath = [[NSBundle mainBundle] pathForResource:@"client" ofType:@"p12"];
        NSData *PKCS12Data = [[NSData alloc] initWithContentsOfFile:thePath];
        CFDataRef inPKCS12Data = (CFDataRef)CFBridgingRetain(PKCS12Data);
        SecIdentityRef identity;
        // 读取p12证书中的内容
        OSStatus result = [self extractP12Data:inPKCS12Data toIdentity:&identity];
        
//        if(result != errSecSuccess){
//            completionHandler(NSURLSessionAuthChallengeCancelAuthenticationChallenge, nil);
//            return;
//        }
        
        SecCertificateRef certificate = NULL;
        SecIdentityCopyCertificate(identity, &certificate);
        
        const void *certs[] = {certificate};
        CFArrayRef certArray = CFArrayCreate(kCFAllocatorDefault, certs, 1, NULL);
        
        credential = [NSURLCredential credentialWithIdentity:identity certificates:(NSArray*)CFBridgingRelease(certArray) persistence:NSURLCredentialPersistencePermanent];
    }
 
    if (credential != nil)
    {
        [challenge.sender useCredential:credential forAuthenticationChallenge:challenge];
    }
    else
    {
        [challenge.sender continueWithoutCredentialForAuthenticationChallenge:challenge];
    }
}
 
- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
{
    [[self client] URLProtocol:self didReceiveResponse:response cacheStoragePolicy:NSURLCacheStorageNotAllowed];
    
}
 
- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
    [_data appendData:data];
}
 
- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
    [[self client] URLProtocol:self didLoadData:_data];
    [[self client] URLProtocolDidFinishLoading:self];
    
}
 
-(void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
    [[self client] URLProtocol:self didFailWithError:error];
    
}
- (OSStatus)extractP12Data:(CFDataRef)inP12Data toIdentity:(SecIdentityRef *)identity {
    OSStatus securityErr = errSecSuccess;
    
    CFStringRef pwd = CFSTR("此处为证书密码");
    const void *keys[] = {kSecImportExportPassphrase};
    const void *values[] = {pwd};
    
    CFDictionaryRef options = CFDictionaryCreate(NULL, keys, values, 1, NULL, NULL);
    CFArrayRef items = CFArrayCreate(NULL, 0, 0, NULL);
    securityErr = SecPKCS12Import(inP12Data, options, &items);
    
    if (securityErr == errSecSuccess) {
        CFDictionaryRef ident = CFArrayGetValueAtIndex(items, 0);
        const void *tmpIdent = NULL;
        tmpIdent = CFDictionaryGetValue(ident, kSecImportItemIdentity);
        *identity = (SecIdentityRef)tmpIdent;
    }
    
    if (options) {
        CFRelease(options);
    }
    
    return securityErr;
}
+(NSMutableURLRequest*)redirectHostInRequset:(NSMutableURLRequest*)request
{
    if ([request.URL host].length == 0) {
        return request;
    }
    
    NSString *originUrlString = [request.URL absoluteString];
    NSLog(@"-----链接导向---------->:%@",originUrlString);
    
    NSString *originHostString = [request.URL host];
    NSRange hostRange = [originUrlString rangeOfString:originHostString];
    if (hostRange.location == NSNotFound) {
        return request;
    }

//此处可以将网页中链接的域名替换为自己的域名,防止劫持.
//    NSString *ip = @"app.xxxx.com";
//     // 替换域名
//    NSString *urlString;
//    if ([originUrlString isEqualToString:@"apis.map.qq.com"]) {
//        urlString=originUrlString;
//    }else if ([originUrlString isEqualToString:@"weibo.com"]){
//        urlString=originUrlString;
//    
//    }else{
//        urlString = [originUrlString stringByReplacingCharactersInRange:hostRange withString:ip];
//    }
    
//    NSURL *url = [NSURL URLWithString:originUrlString];
//    request.URL = url;
    
    return request;
}
@end




```

>OK,完成,解决了uiwebview加载不了图片,css,js等外部资源,同时可以访问微博等第三方https页面.并且对每个https的请求做处理,提升加载速度在5秒以内(安卓的超过10秒).

## 结语

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/53410537](https://blog.csdn.net/luochuanAD/article/details/53410537) 

## 参考

[http://bewithme.iteye.com/blog/1999031](http://bewithme.iteye.com/blog/1999031)
[http://www.cocoachina.com/ios/20160928/17663.html](http://www.cocoachina.com/ios/20160928/17663.html)
[http://www.cocoachina.com/bbs/read.php?tid=1689632](http://www.cocoachina.com/bbs/read.php?tid=1689632)
[http://www.cocoachina.com/ios/20151021/13722.html](http://www.cocoachina.com/ios/20151021/13722.html)
[http://www.jianshu.com/p/7c89b8c5482a](http://www.jianshu.com/p/7c89b8c5482a)








---
layout:     post
title:      centOS7系统下如何导入Python3.6.8
subtitle:   Linux虚拟机
date:       2020-12-25
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- Python 
- Linux

---

## 前言

>最近女朋友遇到一个课题,如何在centOS7系统环境下导入Python3.6 
分析:我的电脑是MacPro,那么我需要安装Linux虚拟机,并且在虚拟机中安装centOS7的系统,然后在centOS7的系统下导入Python3.6.8. 花费了我2天时间,现在整理出来供大家参考.


## 步骤



### 一,下载python3.6.8 

#### 1,导入wget库:命令:

```
yum -y install wget

```

#### 2,导入gcc库:命令:

```
yum -y install gcc
```

#### 3,下载python3.6.8:命令:

```
wget https://www.python.org/ftp/python/3.6.8/Python-3.6.8.tgz
```

< 截图如下:
![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122501.png)



### 二,centOS7关联库的安装 

#### 1,命令:

```
yum -y groupinstall "Development tools"
```

#### 2,命令:

```
yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel
```




### 三,解压,编译,安装python3.6.8

#### 1,解压:命令:

```
tar -zxvf Python-3.6.8.tgz          
```

#### 2,进入解压后的目录:命令

```
cd Python-3.6.8                       
```

#### 3, 编译，安装1: 命令:

```
./configure —prefix=/usr/local/python3
```

#### 4,编译，安装2:命令:

```
make && make install  
```
< 安装完成后,截图如下:
![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122502.png)

### 运行并测试Python3.6.8

#### 1,进入python3目录:命令:

```
cd /usr/local/python3/
```
#### 2,输出当前文件夹目录:命令:

```
ll
```
#### 3,看到bin文件夹,输出此文件夹目录:命令

```
ll bin
```
#### 4,看到python3的文件,运行:命令:

```
bin/python3
```
#### 5,测试python3:代码:

```
print(‘hello world’)
```
#### 6,测试结束,退出:代码:

```
exit()
```
< 测试截图如下:

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122503.png)

## 遇到的问题及解决方案

### 问题1:导入centerOS7系统所依赖的安装包

(例)命令:   yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel xz-devel gdbm-devel db4-devel libpcap-devel

报错:Cannot find a valid baseurl for repo: base/7/x86_64

< 报错截图如下
![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122504.png)

解决方法:
(1), 输入命令:  cd /etc/sysconfig/network-scripts   切换到该文件夹

(2), 输入命令: ls -a   查看该文件夹信息

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122505.png)

(3),输入命令: vi ifcfg-ens33  编辑含有”ifcfg-ens”信息的文件夹

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122506.png)
(注:现在进入到编辑模式，按 i 键进入，然后按往下的方向键，将光标移动到 ONBOOT=no ，然后按往右的方向键，将光标移动到 ONBOOT=no 的 n下面，然后再按 del键。将 no 改成 yes ，也就是 ONBOOT=yes 。然后按 Esc ，再输入 :wq ，保存后退出。)


### 问题2:编译，安装1 时提示需要导入Perl5
命令:
./configure —prefix=/usr/local/python3

< 报错截图:
![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122507.png)

解决方法:(下载perl5)
(1)下载安装包
wget http://www.cpan.org/src/5.0/perl-5.16.1.tar.gz

(2)解压源码包
tar -xzf perl-5.16.1.tar.gz

(3)进入源码目录
cd perl-5.16.1

(4)自定义安装目录
./Configure -des -Dusethreads -Dprefix=/usr/local/perl

(5)下面这三个命令要依次都执行，这是在编译源码
make
make test
make install


(6)查看版本
perl -v

![](https://raw.githubusercontent.com/LuochuanAD/BlogSourceImage/master/BlogSourceImage/BlogSourceImage2020/2020122508.png)

## 参考

Mac安装虚拟机详细步骤:[https://blog.csdn.net/zhangfeidianzi/article/details/103622502](https://blog.csdn.net/zhangfeidianzi/article/details/103622502) 





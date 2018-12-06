---
layout:     post
title:      JavaScript基础之常见的错误
subtitle:   JavaScript
date:       2017-07-25
author:     LuochuanAD
header-img: img/home_blog_background.jpg
catalog: true
tags:
- iOS 
- JavaScript

---

## 前言

>这是JavaScript上书写的常见错误, 我就犯过,在此记录一下


## 错误集


### 一,使用未经定义的变量:

(1) alert(variableValue);   报错:调用了未经定义的变量varibleValue.
(2) function getResult(variableValue){
alert(variableValu); 错误:拼写错误,调用了未经定义的变量variableValu
 }
 
定义一个变量有2种:推荐显示定义变量,对变量的作用域是有影响的.
 (1) var variableValue = 100;    显示定义变量
 (2) variableValue = 100; 隐式定义变量 (注:不推荐这种写法)
 
### 二,区分大小写

```
var variableValue = "love";
if (variableValue == "love"){
alert(variableValue.toUppercase()); //错误:大小写错误, 正确的方法是toUpperCase()
}

```

### 三,缺少大括号(注:代码格式很重要)


```
function getResult(){
var x=1;
var y=2;
if (x <= y){
if (x == y){
if (x == 1){
alert("x and y are equal to 1");
}
}
}    //错误:缺少大括号. 对于这种判断很多的情况下, 代码格式书写规范,是很容易发现错误的  
    

```

### 四,缺少小括号

```

var variableValue = 100; 
if (variableValue + 12)/5 >2){  //报错:缺少小括号
alert("true");
}

```

### 五,赋值(=) 不是相等(==)

```
var variableValue = 100; 
if (variableValue = 99){   //错误:此处应为:相等==
alert("false");
}else{
alert("true");
}

```

### 六,属性和方法名混淆

(1)//获取当前时间的月份是几月
var nowDate = new Date();
alert(nowDate.getMonth);     //错误:获取月份的是个方法名,方法名在调用时都要加(). 
 所以正确的代码是
 alert(nowDate.getMonth());

(2)//获取某个字符串的长度
var variableValue = "love";
alert(variableValue.length());  //错误:length为属性,不应该加().  所以正确的代码是alert(variableValue.length);


### 七,拼接字符串时未使用加号(+)

```
var  myName = "I am ";
var variableValue = "the best man";
variableValue = myName +" " variableValue;        //错误:缺少加号(+).
alert(variableValue);

```

## 结语

这个自定义alertView很简单. 同时我的语言组织的不好,请原谅,这是我2016年来第一次写的博客.见笑了.

博客旧版原文在CSDN上:[https://blog.csdn.net/luochuanAD/article/details/76066604](https://blog.csdn.net/luochuanAD/article/details/76066604) 





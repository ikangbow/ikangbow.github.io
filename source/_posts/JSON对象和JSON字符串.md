---
title: JSON对象和JSON字符串
date: 2020-06-05 18:31:07
tags: java
category: JS
---
# JSON对象

## 对象的概念

对象的属性是可以用： 对象.属性进行调用的。

    var person={"name":"tom","sex":"男"}//json对象
	console.log(person.name);//在控制台输出tom
	alert(typeof(person));//object

以上就是json对象。是一个可以用person.name这种方式进行属性的调用。
第三行代码就是看person的类型为object类型

# JSON字符串

## 字符串，JS中的字符串是单引号或者双引号引起来的。那么json字符串就是如下

    var b = '{"name":"tom","sex":"男"}';//json字符串
	console.log(b);//{"name":"tom","sex":"男"};
	alert(typeof(b));//string

以上b就是一个字符串，是一个string类型

# JSON字符串和JSON对象的转换

## JSON字符串转json对象，调用parse方法；

    var b = '{"name":"tom","sex":"男"}';//json字符串
	console.log(b);//{"name":"tom","sex":"男"};
	alert(typeof(b));//string
	var objb = JSON.parse(b);
	console.log(objb.name);

## JSON对象转JSON字符串

	var person={"name":"tom","sex":"男"}//json对象
	console.log(person.name);//在控制台输出tom
	alert(typeof(person));//object
	var personStr = JSON.stringify(person);
	alert(typeof(personStr));//string
	console.log(personStr);//{"name":"tom","sex":"男"}

## 字符串转json对象

	场景：
		后台返回的person对象上有herf属性，是一个字符串
	href = {'href':'../juvenile/summerCamp/summerCamp.html','isLogined':'1','param':{'activityId':'2141108450038362510'}}

	要解析到herf

	var configData = eval('(' + href + ')');//由字符串转换为JSON对象

	console.log(configData.href)

	


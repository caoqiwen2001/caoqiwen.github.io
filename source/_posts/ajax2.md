---
title: Ajax中几种请求数据方式
date: 2018-01-10 17:54:15
categories:
  - 编程
type: tags
---
### 前言

最近在开发需求的碰到Ajax请求的时候发现请求的数据在后台拿不到数据，故写此篇记录下开发中存在的问题，一般开发中我们用的最多的是post和get请求，对于取数据，像select一般直接get请求就过来，而对于update和delete操作，是对服务器的数据直接操作，所以用post操作比较妥当。
#### get请求
get请求可以是这种写法：
```
	$.ajax({
						type: "GET",
				        url:baseUrl+ "updateOffSys",
				        cache: false,
				        data: {
				        	state:1
				        	name：'lilong'
				        },
				        success: rResult,
				        error: rResult
					});
```
上面这种形式data是json对象，注意不是字符串，发送的时候会默认转成字符串拼接的方式，如：

```
data:"state=1"+"&name=lilong"
```
当然也可以直接通过拼接发送。
后台Spring接收的时候通常可以通过request.getParameter获取，或者也可以函数中直接获取。
#### post请求
对于post请求而言，情况就稍微复杂一点了，对于表单元素来说，它的content-type默认为application/x-www-form-urlencoded，则ajax请求是这样子的：


```
  $.ajax({
            type: "post",
            url: CONFIG.baseUrl + "/audit/editSalesInfo",
            dataType: "json",
            data: data,
            content-type:"application/x-www-form-urlencoded"
            success: function (parameters) {
                var data = parameters;
                if (data["code"] == 0) {
                    layer.alert('审核成功');
                    formReset($("#verifyd_form")[0]);
                } else {
                    layer.alert('审核失败');
                }
            }
        });
```
注意在发送的过程中data的数据还是**json对象**，**不是字符串**其中dataType为json，表示返回到前台的时候数据已经是对象的形式了，不需要在用JSON.parse函数去转换成对象。
此时后台取数据的时候还是可以按照request.getParameter去取数据。  
那么，问题来了，假如我想后台通过springmvc中的@RequestBody来取的时候，前台请求可能是这样子的一种情况：

```
  $.ajax({
            type: "post",
            url: CONFIG.baseUrl + "/audit/editSalesInfo",
            dataType: "json",
            data: JSON.stringfy(data),
            content-type:"application/json"
            success: function (parameters) {
                var data = parameters;
                if (data["code"] == 0) {
                    layer.alert('审核成功');
                    formReset($("#verifyd_form")[0]);
                } else {
                    layer.alert('审核失败');
                }
            }
        });
```
大家注意这个请求和上面这个请求的区别了吗，这个请求是传递一个json字符串，注意是字符串，content-type和data都需要对应上才能正常请求，后台需要加入json-databind的JAR包，并实现对应json转换，否则可能会报错。

#### 总结
对于get请求和post请求，个人建议传过去的data还是用js对象的形式比较好，如果非要采用字符串的形式传给后台，记得后台要加上对应的配置和注解，否则会报错。
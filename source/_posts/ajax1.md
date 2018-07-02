---
title: 使用Ajax创建一个简单的网页交互
date: 2016-09-27 00:28:21
tags:
  - test
  - hexo
categories:
  - 编程
type: tags
---
&emsp;使用它的XMLHttpRequest请求，不需要刷新网页就可以交互数据。比如说用户的注册交互事件，大致可以分为以下几个步骤进行<br>
<li> 创建一个 jsp页面，输入页面元素文档
<li> 采用jqury库中的函数对表单元素进行操作，通过采用post请求将数据转发到post请求。
<li>创建一个serverlt将 数据中转过来之后将数据进行交互，然后再将数据转发到原网页当中去
<br>

### 具体实现代码 ###

	$(function() {
		$(":input[name='username']").change(function() {
			var val = $(this).val();
			val = $.trim(val);
			if (val != "") {
				var url = "${pageContext.request.contextPath}/valiateUserName";
				var args = {
					"username" : val,
					"time" : new Date()
				};
				$.post(url, args, function(data) {
					$("#message").html(data);
				});
			}
		});
	})
<br>
post请求之后的数据直接写入到html中，当然可以看看java的代码是如何实现的：

	List<String>userNames=Arrays.asList("AAA","BBB","CCC");
		String userName=request.getParameter("username");
		System.out.println(userName);
		String result=null;
		if (userNames.contains(userName)) {
			result="<font color='red'>该用户名已经被使用</font>";
		}else{
			result="<font color='black'>该用户名可以使用</font>";
		}
		response.setContentType("text/html; charset=utf-8");
		System.out.println(result);
		response.getWriter().print(result);
通过得到传过来的参数username将数据进行初始化之后，然后对比是否可以注册当前的名字来返回html的文字来作为最终的结果。

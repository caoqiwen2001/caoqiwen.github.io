---
title: Spring中配置bean需要注意的点
date: 2016-09-30 01:40:53
tags:
  - Spring
categories:
    - Spring
type: tags
---
   在Spring中配置通知的时候需要注意以下几个点：
   * 需要的包：将spring中libs中所有的包都包含进去，然后还要配置额外三个包,aopalliance.jar,aspectjrt.jar+aspectjweaver.jar这两个包。否则会出现报错。
   * 在配置对应的文件的时候，需要导入相应的命名空间，如以下配置：
   ```
   <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/context
           http://www.springframework.org/schema/context/spring-context-3.0.xsd
           http://www.springframework.org/schema/aop
           http://www.springframework.org/schema/aop/spring-aop-3.0.xsd">
	<context:component-scan base-package="com.caoqiwen.spring.aop" >
        </context:component-scan>
        <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
 </beans>

 ```
 我配置了Context和aop的空间，特别注意的是在schmaLocation的配置中需要和上面的配置命名空间的顺序统一，否则也可能会报错。

*  通过注解来获取对应类的实例方法

```
package com.caoqiwen.spring.aop;
import org.springframework.stereotype.Component;

@Component("arithmeticCalculator")
public class  ArithmeticCalculatorImpl implements ArithmeticCalculator {

	@Override
	public int add(int x, int y) {
		// TODO Auto-generated method stub
		//return 0;
		int result=x+y;
		return result;
	}

	@Override
	public int sub(int x, int y) {
		// TODO Auto-generated method stub
		int result=x-y;
		return result;
	}

	@Override
	public int mul(int x, int y) {
		// TODO Auto-generated method stub
		int result=x*y;
		return result;
	}

	@Override
	public int div(int x, int y) {
		// TODO Auto-generated method stub
		int result=x/y;
		return result;
	}
}
```
会根据配置去扫描com.caoqiwen.spring.aop命名空间下面的所有类

### 关于切面优先级的问题 ###
&emsp; 对于同一个切面，可以在类的前面注释一个order优先级，优先级，如：
```
@Order(2)
@Component
@Aspect
public class DesperateAspect {

	@Before("execution(public int com.caoqiwen.spring.aop.ArithmeticCalculator.*(..))")
	public void beforeMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		Object[] args = joinPoint.getArgs();
		System.out.println("the DesperateAspect" + methodName + "is start" + Arrays.asList(args));
	}
}

```
如果优先级越低，则该方法会优先执行。

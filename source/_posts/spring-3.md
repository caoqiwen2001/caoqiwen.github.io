---
title:  Spring生命周期详解
date: 2018-03-09 16:30:23
tags:
  - Spring
categories:
    - Spring
type: tags
---
### Spring的生命周期分析
研究Spring的意义在于知道它的整个生命周期是怎么样一步一步从一个配置文件如何生成我们所需要的bean的。作为一个中高级开发人员，如果只懂一些基本的配置文件，那是肯定不及格的，只有知道了它的原理，在遇到问题才能找到处理方法。
### 基础实例

```
public class Car implements BeanFactoryAware, BeanNameAware, InitializingBean, DisposableBean, ApplicationContextAware {

    private String brand;
    private String color;
    private int maxSpeed;

    private BeanFactory beanFactory;
    private String beanName;

    public Car() {
        System.out.println("调用Car()构造函数");
    }

    public void setBrand(String brand) {
        System.out.println("调用setBrand()设置属性");
        this.brand = brand;
    }

    public void introduce() {
        System.out.println("brand:" + brand + ";color:" + color + ";maxSpeed:" + maxSpeed);
    }

    /**
     * BeanFactoryAware 接口方法
     */
    public void setBeanFactory(BeanFactory beanFactory) throws BeansException {
        System.out.println("调用BeanFactoryAware.setBeanFactory()");
        this.beanFactory = beanFactory;
    }

    /**
     * BeanNameAware接口方法
     */
    public void setBeanName(String name) {
        System.out.println("调用BeanNameAware.setBeanName()");
        this.beanName = name;
    }

    /**
     * DisposableBean接口
     */
    public void destroy() throws Exception {
        System.out.println("调用DisposableBean.destroy()");
    }

    /**
     * InitializingBean接口
     */
    public void afterPropertiesSet() throws Exception {
        System.out.println("调用InitializingBean.afterPropertiesSet()");
    }

    /**
     * ApplicationContextAware 接口
     */
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("调用ApplicationContextAware.setApplicationContext()");
    }

    /**
     * 通过<bean>的init-method属性指定的初始化方法
     */
    public void myInit() {
        System.out.println("调用init-method指定的myInit()，将maxSpeed设置为240");
        this.maxSpeed = 240;
        System.out.println("调用init-method指定的myInit()，将color设置为绿色");
        this.color="绿色";

    }

    /**
     * 通过<bean>的destroy-method属性指定的初始化方法
     */
    public void myDestroy() {
        System.out.println("调用destroy-method指定的myDestroy()");
    }

    //--------------getter and setter--------------------------------

    public String getBrand() {
        return brand;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public int getMaxSpeed() {
        return maxSpeed;
    }

    public void setMaxSpeed(int maxSpeed) {
        this.maxSpeed = maxSpeed;
    }

```
定义了一个需要实现的bean文件，实现的接口包括：BeanFactoryAware，BeanNameAware，InitializingBean，ApplicationContextAware。

```
public class MyBeanFactoryPostProcessor implements BeanFactoryPostProcessor {
    //对car<bean>的brand属性配置信息进行“偷梁换柱”的加工操作
    public void postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) throws BeansException {
        BeanDefinition bd = beanFactory.getBeanDefinition("car");
        bd.getPropertyValues().addPropertyValue("brand", "奇瑞QQ");
        System.out.println("调用BeanFactoryPostProcessor.postProcessBeanFactory()");
    }
}
```
定义了一个工厂后置管理器，该方法在加载配置文件初始化之后修改配置文件中的数据。


```
public class MyBeanPostProcessor implements BeanPostProcessor {
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("car")) {
            Car car = (Car) bean;
            if (car.getColor() == null) {
                System.out.println("调用BeanPostProcessor.postProcessBeforeInitialization()，color为空，设置为默认黄色");
                car.setColor("黄色");
            }
        }
        return bean;
    }

    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        if (beanName.equals("car")) {
            Car car = (Car) bean;
            if (car.getMaxSpeed() >= 200) {
                System.out.println("调用BeanPostProcessor.postProcessAfterInitialization()，将maxSpeed调整为100");
                car.setMaxSpeed(100);
            }
        }
        return bean;
    }
}
```
定义了一个MyBeanPostProcessor实现了一个BeanPostProcessor接口，该接口实现了postProcessBeforeInitialization方法，postProcessAfterInitialization接口。  
接着配置下xml文件：

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="car" class="com.brianway.learning.spring.ioc.beanfactory.Car"
          init-method="myInit"
          destroy-method="myDestroy"
          p:brand="红旗CA72"
          p:maxSpeed="200"
          scope="singleton"/>

    <bean id="myBeanPostProcessor" class="com.brianway.learning.spring.ioc.context.MyBeanPostProcessor"/>

    <bean id="myBeanFactotyPostProcessor" class="com.brianway.learning.spring.ioc.context.MyBeanFactoryPostProcessor"/>

</beans>
```
写一段测试代码来进行测试：

```
 private String beanId = "car";
  String xmlPath = "com/brianway/learning/spring/ioc/context/beans-singleton.xml";
        BeanLifeCycle.lifeCycleInApplicationContext(xmlPath, beanId);
 
```
结果输出如下：

```
调用BeanFactoryPostProcessor.postProcessBeanFactory()
调用Car()构造函数
调用setBrand()设置属性
调用BeanNameAware.setBeanName()
调用BeanFactoryAware.setBeanFactory()
调用ApplicationContextAware.setApplicationContext()
调用BeanPostProcessor.postProcessBeforeInitialization()，color为空，设置为默认黄色
调用InitializingBean.afterPropertiesSet()
调用init-method指定的myInit()，将maxSpeed设置为240
调用init-method指定的myInit()，将color设置为绿色
调用BeanPostProcessor.postProcessAfterInitialization()，将maxSpeed调整为100
brand:奇瑞QQ;color:绿色;maxSpeed:100
第二次从容器中获取car
brand:奇瑞QQ;color:红色;maxSpeed:100
car1==car2：true
```
1. 从结果输出可以看到，首先看到调用postProcessBeanFactory方法，这个方法只调用一次。
2. 接着初始化了Car中的构造函数，调用setBrand方法设置相关属性。
3. 下一步调用BeanPostProcessor.postProcessBeforeInitialization()，它是针对每个bean初始化之前进行的操作。
4. 接着调用afterPropertiesSet方法和init-method方法。
5. 下一步调用postProcessAfterInitialization方法。
### 总结
Spring生命周期中的周期清楚之后，接下来的篇章主要对bean的产生和bean的初始化进行详细分析。




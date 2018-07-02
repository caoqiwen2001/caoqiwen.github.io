---
title: Java设计模式-代理模式
date: 2017-12-15 16:43:06
tags:
  - Java
categories:
  - 设计模式
---
### 代理模式
代理模式的定义很简单，给某一个对象提供一个代理对象，由代理对象控制原对象的引用。
###  代理模式需要的几个元素
- 抽象对象角色
- 目标对象角色
- 代理对象角色

---
### 具体事例
通过模拟登陆网站来实现代理模式，如网易，像大型网站，采用分布式架构，我们访问的时候通过Nginx，用来做负载均衡。最终访问的路径就是


```
graph LR 
A--用户访问网易-->B
B--访问代理服务器-->C
C--最终服务器-->D
```
首先需要一个服务器接口，获取网站的数据。

```

public interface Server {
    public String getPageTitle(String url);
}

```
接着需要实现网易服务器的具体实现，获取网易的标题

```
public class WYProxy implements Server {
    @Override
    public String getPageTitle(String url) {
        if ("http://www.163.com/".equals(url)) {
            return "网易首页";
        }
        return "无网站首页";
    }
}
```
然而需要通过代理服务器才能访问到该网站，如何访问呢，需要创建一个代理对象。

```
package com.caoqiwen.learning.design.example5;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

/**
 * Created by caoqiwen on 2017/12/15.
 * 模拟Ngix反向代理
 */
public class NgixProxy implements Server {

    private static final List<String> WY_SERVER_ADDRESS = new ArrayList<String>() {{
        add("192.168.0.1");
        add("192.168.0.2");
        add("192.168.0.3");
    }};

    public  NgixProxy(Server server){
        this.server=server;
    }

    private Server server;

    public Server getServer() {
        return server;
    }

    public void setServer(Server server) {
        this.server = server;
    }

    @Override
    public String getPageTitle(String url) {
        String remoteIP = UUID.randomUUID().toString();
        int index = Math.abs(remoteIP.hashCode()) % WY_SERVER_ADDRESS.size();
        String realIP = WY_SERVER_ADDRESS.get(index);
        return "网站" + server.getPageTitle(url) + "[来源ip" + realIP + "]";
    }
}

```
远程IP通过UUID然后对函数取余得到具体ip地址，而实际的网站通过内部server对象获取。
### 静态代理的缺点
采用静态模式是可以很好的解决代码耦合的问题，但是也存在一些问题。  
- 只能服务于Server接口，不能复用，只能新增接口或者新增接口类的实现，会造成类的膨胀。
- Server接口新增一个接口，代理类也需要跟着实现该接口，无法实现类的复用。

---
### JDK中动态代理的实现

```
public class NginxInvocationHandler  implements InvocationHandler{
    private static final List<String> WY_SERVER_ADDRESS = new ArrayList<String>() {{
        add("192.168.0.1");
        add("192.168.0.2");
        add("192.168.0.3");
    }};
    private  Object object;

    public NginxInvocationHandler(Object object){
        this.object=object;
    }
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        String remoteIP= UUID.randomUUID().toString();
        int index = Math.abs(remoteIP.hashCode()) % WY_SERVER_ADDRESS.size();
        String realIP = WY_SERVER_ADDRESS.get(index);
        StringBuffer stringBuffer=new StringBuffer();
        stringBuffer.append("页面标题:");
        stringBuffer.append(method.invoke(object,args));
        stringBuffer.append("[来源地址]:");
        stringBuffer.append(realIP);
        return stringBuffer.toString();
    }
}
```
只需要实现InvocationHandler这个接口即可，通过newProxyInstance方法产生实例对象。也是Spring中实现AOP的精华之处。

```
 Server wyServer = new WYServer();
        InvocationHandler invocationHandler = new NginxInvocationHandler(wyServer);
        Server server = (Server) Proxy.newProxyInstance(DynamicProxyTest.class.getClassLoader(), new Class[]{Server.class}, invocationHandler);
        System.out.println(server.getPageTitle("http://www.163.com/"));
```
动态代理的缺点在于只能产生接口的实例对象，如果需要实例化一个类的动态对象，我们需要用CGLIB来实现。



###  CGLIB实现动态代理
上面说了对于没有实现接口类如何动态生成对应的对象，我们采用CGLIB就可以实现，在AOP中也用到了CGLIB的动态代理方法。首先来看CGLIB的实现方法。

```
public class Dao {

    public void update() {
        System.out.println("update dao has begined");
    }

    public void select() {
        System.out.println("select dao has  begined");
    }
}
```
需要定义一个实现类，而不是接口。

```
public class DaoProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
        System.out.println("before method invoke");
        Object object = methodProxy.invokeSuper(o, objects);
        System.out.println("after method invoke");
        return object;
    }
}
```
通过继承实现MethodInterceptor方法实现，可以在intercept中加入我们需要的业务逻辑。

```
DaoProxy daoProxy = new DaoProxy();
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(Dao.class);
        enhancer.setCallback(daoProxy);
        Dao dao = (Dao) enhancer.create();
        dao.update();
        dao.select();
```
最后一步产生我们需要的类。
#### 需要注意的事项
在maven中需要添加CGLIB的jar文件，还有asm的jar包，否则会提示报错，具体需要添加如下：

```
            <dependency>
                <groupId>cglib</groupId>
                <artifactId>cglib</artifactId>
                <version>3.2.1</version>
            </dependency>

            <dependency>
                <groupId>org.ow2.asm</groupId>
                <artifactId>asm</artifactId>
                <version>5.0.4</version>
            </dependency>
```
















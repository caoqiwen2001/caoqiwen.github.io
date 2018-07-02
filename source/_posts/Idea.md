---
title: idea神器配置和使用
date: 2016-11-12 00:03:54
tags:
  - hexo
categories:
  - Java
type: tags
---
# 为什么使用idea
最近公司全部用idea开发了，原来用eclipse开发的时候发现有时候莫名其妙的报错，而idea带给我的是另一番体验，自动的pom管理jar包，断点调试也是各种爽，单步调试每次的值不要鼠标停留才会显示出来，比eclipse爽多了，所以推荐大家用idea。
# 如何配置对应的maven项目
首先需要安装maven对应的包，然后在在环境中配置对应的系统变量，然后就可以在maven中进行配置了。然后就是一堆很多的配置文件，这个是个体力活，但是配置完之后就是一劳永逸的事情了。
# 配置选项
去官网下载对应的安装包目录，假设我们的依赖包都已经全部下载完成之后，我们可以要设置tomcat和run Configurations等设置

1. 按下快捷键ctrl+alt+s,打开setting界面，设置maven的相关选项。
 ![1.jpg](https://ooo.0o0.ooo/2016/11/11/5825e7f4224b6.jpg)将对应的依赖包放在特定的盘。
2、按下快捷键ctrl+alt+shift+S,开始设置项目的结构设置。
![2.jpg](https://ooo.0o0.ooo/2016/11/11/5825e97cb1e7e.jpg)
会看到设置项目的project选项，一个一个分别设置上去。等这两步设置成功了。就可以开始设置我们的tomcat了。
3、打开Run/Debug Configurations 选项，新建一个tomcat，设置对应的名字和tomcat所在的位置，配置好Deployment。
![3.jpg](https://ooo.0o0.ooo/2016/11/11/5825eaf2e8d43.jpg)
注意要配置对应的exploded包才行
项目如果没有错误，会提醒我们配置对应包，在Application context中要去配置对应的包名，就可以开启tomcat开使我们的代码之旅了。

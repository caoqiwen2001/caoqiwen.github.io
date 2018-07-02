---
title: 如何用hexo搭建个人博客
date: 2016-08-12T02:05:39.000Z
tags:
  - test
  - hexo
categories:
  - 编程
type: tags
---

终于搭建了自己的个人博客，期间遇到一些一些问题，不过都解决了，网上关于这方面的教程也挺多的，主要说下要预防可能发生错误的点
- 配置文本个人deploy的时候一定要记住要有空格，在type的后面记得要空一格再输入，可以选择使用notepad++勾选空格显示，否则hexo-d是不会成功的，也就是说代码push到github上是不会成功的，导致打开自己网站的时候出现404的错误。如下图所示<div align="center">
    <img src="http://ww1.sinaimg.cn/large/b8b72f14gw1f6uvuazme4j20fu04pglk.jpg" alt="Alt text">
<p align="left">如果省去上面的部分代码github就连接不上了</p>
  </div>
- 提交成功了但是也报404的错误，一度怀疑自己是不是哪里配置错误了，后来才发现自己的邮箱没有验证成功，github发了很多封邮件都当垃圾处理了，验证完成之后输入自己的网址就成功了，如我的网址是[caoqiwen2001.github.io](https://caoqiwen2001.github.io/)<br><br>
### 常用的指令输入
 *  `hexo-clean` 清除原有的缓存
 *  `hexo-g`  本地生成代码，将md生成对应的静态index文件。
 * `hexo-s` 本地可以预览，直接local：4000可以查看自己写的文章内容
 * `hexo-d`直接部署到github，发布后可以输入自己的网址查看内容。

 &nbsp;&nbsp;&nbsp;&nbsp;上面这几个常用的指令已经能够满足我们日常的写作习惯了，然后用上牛逼的Markdown语法更是非常简洁和迅速。个人感觉非常不错。
### 下面是可以参考的一些博客资源
1. [使用Hexo搭建博客（三），博客配置、主题和写作](http://www.jianshu.com/p/db7e64d86067)
2. [GitHub Pages + Hexo搭建博客](http://crazymilk.github.io/2015/12/28/GitHub-Pages-Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/#more)
3. [Hexo搭建Github静态博客](http://www.cnblogs.com/zhcncn/p/4097881.html)

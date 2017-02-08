---
layout: post
title: "HTTPS改造"
date: 2017-02-07 17:50:04 +0800
comments: true
categories: 
---

参考

[关于启用 HTTPS 的一些经验分享](https://imququ.com/post/sth-about-switch-to-https.html#toc-3)

[HTTPS 改造初探](https://segmentfault.com/a/1190000005950801)

##为何从http改造成https

<!--more-->

直接原因：iOS APP 强制使用HTTPS

间接原因：

* HTTPS 相比HTTP 更加安全；
* HTTPS 是未来的趋势


## https改造知识

何为Mixed Content

HTTPS网页中加载的HTTP资源称之为Mixed Content（混合内容），不同浏览器对这些内容处理规则不一致。

W3C Mixed Content 规范，将其分为Optionally-blockable和Blockable 两类，顾名思义前者可以被浏览器加载，后者不能被加载

Optionally-blockable 资源主要包括：
&lt;img&gt; &lt;video&gt; &lt;audio&gt; &lt;source&gt;等标签，除此之外都是Blockable的.

但浏览器的区别可能会不支持上述规范。

最佳实践：HTTPS页面中不要出现Mixed Content 资源。


## https改造过程

具体改造主要涉及到前端、后端、运维。

### 后端改造

由于API层不暴露在公网中，是通过RPC调用的，还是采用HTTP进行调用；

涉及到的改造主要是代码层面的修改包含：

* CDN 配置由HTTP 修改为HTTPS (前提CDN支持HTTPS)
* 云存储OSS 配置由HTTP 修改为HTTPS
* 文档预览服务 配置由HTTP 修改为HTTPS
* 数据库富文本中的相关HTTP 替换为HTTPS
* 其他第三方服务HTTP 替换为HTTPS

### 前端改造

主要是修改显式写明http的地方，采用 // 访问，让浏览器自动适配


### 运维配置

运维配置由专门的运维人员进行配置，只知道个大概，有些优化也不知道做了没有

首先需要购买HTTPS 证书，我们买的是Go Daddy 的证书，开始买的是
证书只支持二级域名，我们有服务配置了三级域名，只能重新购买，需要注意。

由于CDN 使用我们自己的域名进行访问，我们的方式比较简单直接把锅甩给了CDN,把我们的证书给他们让他们进行了配置；上面的文章说可以通过 Keyless SSL解决

在 HTTPS 性能上，可以做的几个优化：

* 开启 SPDY 和 HTTP2 支持，借助多路复用提升传输性能。
* 开启会话复用，减少 TLS 握手的时间与性能消耗。
* 开启 HSTS，强制 HTTPS 访问，减少与服务端的交互。
* 配置完整的证书链，离线完成证书的链式认证。
* 开启 OCSP stapling，离线完成证书在线状态检测。
* 进行域名收敛









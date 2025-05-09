---
title: burpsuite基础
date: '2025-05-05 09:39:16'
updated: '2025-05-09 09:33:57'
permalink: /post/burpsuite-basics-1defgi.html
comments: true
toc: true
---



# burpsuite基础

## brupsuite使用笔记

---

按照文档里的操作使用

name:marin

password:D;9Ar[]FdR%ot%rjr3z;c")&}zXf=J4T

### 拦截HTTP流量

---

![](https://pic.imgdb.cn/item/66f5661ff21886ccc013ab90.png)

点击 **代理 &gt; 拦截 &gt; 启用拦截**
打开burp的浏览器，访问`https://portswigger.net`
就可以看到拦截的请求，并且网页是加载不了，单击**放行**，就会发送被拦截的请求，页面就能加载了

![](https://pic.imgdb.cn/item/66f5666cf21886ccc013f7b5.png)

由于流量器会发送大量请求，因此不会想要拦截每一个请求，关闭拦截

**代理 &gt; HTTP历史记录**可以查看所有HTTP流量的记录，即便没有开启拦截

### 修改HTTP请求

---

用内置浏览器打开他给的url，进入购物网站，选择商品

回到burp，打开拦截，回到浏览器点击加入购物车

然后再回到burp，可以看到拦截了一个请求

![](https://pic.imgdb.cn/item/66fce370f21886ccc0c68793.png)

修改最下面的请求体的`price`

![](https://pic.imgdb.cn/item/66fce3c7f21886ccc0c6cbef.png)

在旁边也可以看到具体的参数

![](https://pic.imgdb.cn/item/66fce3e6f21886ccc0c6e410.png)

之后再放行，回到购物车就会发现价格已经改变了

![](https://pic.imgdb.cn/item/66fce41cf21886ccc0c70a4b.png)

### 设置目标范围

---

目标范围是为了告诉burp要测试的url和主机，从而过滤掉浏览器和其他网站

在 **代理** > **HTTP历史记录** 中可以查看到所有浏览器的请求

在 **目标** > **站点地图** 的左侧面板中可以看到浏览器交互的主机

![](https://pic.imgdb.cn/item/66fce5e7f21886ccc0c89a95.png)

右键需要的主机，点击 **添加到范围** 即可设置目标范围

![](https://pic.imgdb.cn/item/66fce61af21886ccc0c8bfed.png)

回到HTTP历史记录，点击上方的过滤器

![](https://pic.imgdb.cn/item/66fce64ef21886ccc0c8e85c.png)

选择 **仅显示范围内的条目** ，就可以只看到目标的请求了

![](https://pic.imgdb.cn/item/6707d9a8d29ded1a8cde3076.png)

### 使用 burp repeater 重新发送请求

---

右击一个请求，选择 **发送到repeater**

![](https://pic.imgdb.cn/item/6707d9b6d29ded1a8cde3ca1.png)

在 **中继器** 中可以查看到请求，修改参数后发送就可以看到响应了，发送右边的箭头就可以查看之前的请求

![](https://pic.imgdb.cn/item/6707d9e4d29ded1a8cde6700.png)

在这个例子里，在productID中输入的参数如果为字符串，就会报错，获得版本号

![](https://pic.imgdb.cn/item/6707d9f0d29ded1a8cde7043.png)

### 运行第一次扫描

---

在 **仪表盘** > **新建扫描** > **web app scan** 中

![](https://pic.imgdb.cn/item/6707d9fed29ded1a8cde7a17.png)

![](https://pic.imgdb.cn/item/6707da04d29ded1a8cde7fe3.png)

写入要扫描的网站，在扫描设定中因为不懂就先选择了预设的

![](https://pic.imgdb.cn/item/6707da10d29ded1a8cde896b.png)

在站点地图中就可以看到发现的内容

![](https://pic.imgdb.cn/item/6707da1ad29ded1a8cde9056.png)

在任务中选择扫描，就可以看到相关的内容，出现什么问题

![](https://pic.imgdb.cn/item/6707da2ad29ded1a8cde9f8f.png)

然后在 **目标** > **站点地图** 右键需要的点击问题，该主机的问题报告，然后一直点下一步就能获得报告

![](https://pic.imgdb.cn/item/6707da31d29ded1a8cdea4f8.png)

### 添加初始测试范围

---

在 **settings** 中的 **scope** 中可以设置要包括和排除的url

![](https://pic.imgdb.cn/item/6707d8d5d29ded1a8cdd7b1e.png)

### 自动发现内容

---

在站点地图右键，选择**相关工具** > **发现内容**

![](https://pic.imgdb.cn/item/6707da41d29ded1a8cdeb4a0.png)

弹出的窗口选择会话并未运行

![](https://pic.imgdb.cn/item/6707da4cd29ded1a8cdebeea.png)

### 发现子域

---

右键要发现的url，选择发送到intruder

![](https://pic.imgdb.cn/item/6707d970d29ded1a8cddfd01.png)

在payload配置中输入需要的url，并在需要的位置输入占位符，右侧选择添加payload位置

![](https://pic.imgdb.cn/item/6707d956d29ded1a8cdde7e8.png)

在上方选择payload选项,在payload设置中选择需要的字典，就可以开始攻击，在页面中寻找和其他请求有不一致的url

![](https://pic.imgdb.cn/item/6708c9fdd29ded1a8c8610a8.png)

### Target分析器

---

在站点地图中，右键目标主机，选择相关工具 > 分析目标

![](https://pic.imgdb.cn/item/6708ca04d29ded1a8c86156d.png)

在这个页面即可看到静态url和动态url的数量，了解目标的大小

![](https://pic.imgdb.cn/item/6708ca0ad29ded1a8c861a7b.png)

在参数页面中可以选择参数，并查看有关的所有url

![](https://pic.imgdb.cn/item/6708ca18d29ded1a8c862434.png)

在站点地图中，查看齿轮图标的也可以查看动态提交的

![](https://pic.imgdb.cn/item/6708ca1cd29ded1a8c8627b0.png)

### 识别高风险功能

---

在站点地图中，有带颜色圈圈的就是有风险的，点击可以查看对应风险

![](https://pic.imgdb.cn/item/6708ca1cd29ded1a8c8627b0.png)

在代理中打开拦截，设置拦截的规则，在**请求拦截规则**选择拦截基于以下的规则

![](https://pic.imgdb.cn/item/6708ca35d29ded1a8c86398e.png)

### 确定支持的HTTP方法

---

#### 单个端点支持的方法

由于配置错误，有些网站可能可以通过非标准的方法进行访问，要确定支持的HTTP方法就可以判断哪些可以访问

将要识别的网站发送到**intruder**中，在payload中设置简单列表

![](https://pic.imgdb.cn/item/66fe02ba0a206445e383d1d9.png)

在请求方式上加入payload位置

![](https://pic.imgdb.cn/item/66fe03620a206445e384577a.png)

同时在payload设置中选择HTTP动词

![](https://pic.imgdb.cn/item/6708ca47d29ded1a8c8644d7.png)

攻击就可以查看哪些方法可以被响应

![](https://pic.imgdb.cn/item/6708ca51d29ded1a8c864e38.png)

#### 多个端点支持的方法

攻击方式选择多个payload集合

![](https://pic.imgdb.cn/item/6708ca57d29ded1a8c86524e.png)

选择请求方式和目标路径作为payload位置

![](https://pic.imgdb.cn/item/6708ca5dd29ded1a8c8656ce.png)

在设置中，通过切换payload集对他们进行不同的设置，在这里的2中，要输入所有的url，在站点地图中，选择主机，右键选择复制所有url，然后在这里粘贴

![](https://pic.imgdb.cn/item/6708ca65d29ded1a8c865d3d.png)

最后的payload编码可以去掉，防止对斜杠字符进行编码

![](https://pic.imgdb.cn/item/6708ca6cd29ded1a8c8662d7.png)

### 检查隐藏的输入

---

隐藏输入就是HTTP标头，cookie或参数，会影响站点响应的方式

最开始准备，在拓展中下载一个 Param Miner BApp

![](https://pic.imgdb.cn/item/6708ca74d29ded1a8c866761.png)

在站点地图中，对要查找的主机右键，选择 **拓展** > **Param Miner** 然后根据需求选择要什么参数

![](https://pic.imgdb.cn/item/6708ca7bd29ded1a8c866c22.png)

最后可以在**拓展**中查看输出

![](https://pic.imgdb.cn/item/6708ca83d29ded1a8c8670c9.png)

### 编码工具

---

选择需要编码的字符串，右键，选择发送到**Decoder**

![](https://pic.imgdb.cn/item/6708ca89d29ded1a8c8674ec.png)

转到编码工具就可以使用了

![](https://pic.imgdb.cn/item/6708ca96d29ded1a8c868037.png)

### 测试身份验证机制

---

这个应该开始涉及到了OWASP中的内容，这个像是A07
给了一个lab，并且给了用户名和密码，相当于泄露了

我直接使用Intruder攻击，发现给的都是错误的，只有一个302的，查看教程，在开始攻击之前，在攻击页面的设置中有个**检索-提取**，可以从响应中根据开头结尾提取信息

![](https://pic.imgdb.cn/item/6708caa9d29ded1a8c868f8e.png)

然后就会发现au为用户名的响应会少一个`.`

![](https://pic.imgdb.cn/item/6708cabad29ded1a8c869e66.png)

然后都丢到对比里面查看，发现好像是多了注释的前半段

![](https://pic.imgdb.cn/item/6708cabfd29ded1a8c86a1f0.png)

蚌不住了，原来一开始的那个302的请求就是正确的用户密码，没重定向orz

这个实验室的正确思路是，先只使用用户名，通过缺少一个`.`确认用户名，再通过用户名攻击密码，最后找到这个302的响应，我一开始就直接两个一起攻击

这个攻击叫做撞库攻击，在后面有，确定就是A07 身份验证和身份识别失效

### 测试会话管理机制

---

这个是有关seesion token的，~~感觉还是和A07有关~~ 和A01相关的orz

跟着教程走，先登录，然后拿到cookie，将cookie发送到**Sequencer**，在里面点击实时捕捉，将会得到分析结果，虽然看不懂

![](https://pic.imgdb.cn/item/6708cacfd29ded1a8c86af21.png)

#### 使用burp生成CSRF Poc

这个是A01的一个漏洞，一开始做的时候是蒙的

然后详细查了以下CSRF的实现

> **CSRF**：
> 用户首先访问了A网站，并且在A网站上登录了，在客户端产生了cookie，然后用户在没有登出的情况下登录了危险的B网站，B网站向A网站发送了一个请求，带上了用户的cookie，导致A网站确认这是用户的请求，达成了模拟用户操作的目的
>
> **burp CSRF POC**：
> 就是burp模拟危险B网站，生成了一个HTML页面，进入的时候指向了修改的页面，发送请求

接着是这个实验室，目的就是修改邮箱，在扫描可以发现这里有这个CSRF漏洞

![](https://pic.imgdb.cn/item/6708cb6bd29ded1a8c872b26.png)

选择有可能被攻击的请求，在相关工具里生成CSRF Poc

![](https://pic.imgdb.cn/item/6708cb02d29ded1a8c86d7eb.png)

在这个页面就是brup生成了一个请求的页面，复制url到浏览器打开，就会修改邮箱

![](https://pic.imgdb.cn/item/6708cb20d29ded1a8c86ef76.png)

正常的请求的referer是从登录的页面发送过去的，而CSRF发送的referer是从burp过去的（这个是我自己感觉的，不知道对不对orz

![](https://pic.imgdb.cn/item/6708cb26d29ded1a8c86f428.png)

![](https://pic.imgdb.cn/item/6708cb2cd29ded1a8c86f868.png)

#### 使用JWT

首先在拓展中安装**JWT editor**

> OWASP记了JWT是一种令牌，但是没记JWT长什么样
>
> JWT分为三个部分
> 页眉（header）、有效载荷（payload）、签名（signature）
> 每个部分之间用`.`进行分割
>
> * Header：通常由令牌类型`typ`、使用的签名算法`alg`组成
> * Payload：包含了声明Claims,有三种
> * 已注册的声明（registered claims）：是预定义的，包含`iss(发行人)`、`exp(到期时间)`、`sub(主题)`、`aud(受众)`
> * 公共声明（public claims）：是使用JWT的人随意定义，但是为避免冲突，应在IANA JSON Web Token注册表中定义
> * 私人声明（private claims）：是自定义声明
> * Signature：是使用header中的算法加密的
>
> 一个完整的JWT的header和payload是进行了**base64URL**编码

安装了拓展之后，会用绿色自动标记了有JWT的请求

![](https://pic.imgdb.cn/item/6708cb62d29ded1a8c872309.png)

在inspector中可以自动解码，发现JWT就在cookie里

![](https://pic.imgdb.cn/item/6708cb75d29ded1a8c873351.png)

带有JWT的请求发送到repeater，在这里选择JSON WEB TOKEN中的报头和payload中修改JWT

![](https://pic.imgdb.cn/item/6708cb80d29ded1a8c873b86.png)

然后在JWT Editor中添加一个密钥，然后选择回去点击**sign**

![](https://pic.imgdb.cn/item/6708cb86d29ded1a8c874152.png)

然后到这个实验室的要求是进入管理面板，删除一个叫carlos的用户

要进入这个管理面板就要将请求的页面改到`/admin`，修改用户，这里我一开是不知道用户名应该是啥，随便发了个请求，发现这个的提示的用户名

![](https://pic.imgdb.cn/item/6708cb8cd29ded1a8c8746b2.png)

然后修改JWT用户名，发送过去查看发现了管理员面板

![](https://pic.imgdb.cn/item/6708cb92d29ded1a8c874b61.png)

```html
<a href="/admin/delete?username=carlos">Delete</a>
```

找到这个href，向这个href发送请求（一开始这个302我还以为错了，再访问一次发现已经删了

![](https://pic.imgdb.cn/item/6708cb98d29ded1a8c875144.png)

### 测试访问控制

就是为了测试是否访问控制是否正确，这个就是A01

#### 测试权限提升

首先登录管理员的身份，注销，然后登录普通用户，回到burp，复制普通用户的cookie，替换管理员的cookie向控制面板发送请求，得到无法查看

![](https://pic.imgdb.cn/item/6708cbb4d29ded1a8c8764a4.png)

##### 在整个站点进行测试

创建一个会话处理规则， 在设置中**session** > **会话处理规则** 在范围中只选择目标，然后选择自定义范围，添加目标url

![](https://pic.imgdb.cn/item/6708cbbbd29ded1a8c876a92.png)

![](https://pic.imgdb.cn/item/6708cbd0d29ded1a8c8779a6.png)

回到详细信息，选择添加一个规则，将session的值给为之前复制的低权限的值

然后保存，回到站点地图，选择对比站点地图

![](https://pic.imgdb.cn/item/6708cbe2d29ded1a8c878684.png)

然后选择使用当前站点地图，选择仅使用选定的分支，然后一直使用默认设置

就可以查看高权限和低权限不同的请求

![](https://pic.imgdb.cn/item/6708cbf3d29ded1a8c879541.png)

##### 实验：可以规避基于方法的访问控制

用管理员身份登录，访问管理面板，提升一个用户的权限，然后在burp查看该请求，发送到中继器，回到站点，登录普通用户，查看普通用户的cookie，然后将提权的cookie改为普通用户的cookie

得到未授权

![](https://pic.imgdb.cn/item/6708cbf9d29ded1a8c879a07.png)

将请求方法改为postx（这几步不知道为啥orz

![](https://pic.imgdb.cn/item/6708cc00d29ded1a8c87a055.png)

然后右键，选择修改请求方法，将用户名改为普通权限的用户名，成功修改

![](https://pic.imgdb.cn/item/6708cc07d29ded1a8c87a536.png)

![](https://pic.imgdb.cn/item/67065854d29ded1a8c9788d6.png)

#### 测试水平访问控制

网站设置不当，导致用户可以访问本应仅对其他用户可用的数据和功能（其实操作和上面都差不多

先登录两个用户，然后找到第一个用户确定cookie的请求，发送到中继器，然后找到第二个用户的cookie，替换原cookie然后发送请求

![](https://pic.imgdb.cn/item/67065847d29ded1a8c977f4e.png)

#### 测试IDOR

> IDOR(不安全的直接对象参考):网站中有很多变量，例如’id‘，’uid‘，这些值通常被视为HTTP参数，攻击者可以通过更改这些值来访问，编辑或删除其他用户的对象

这里使用一个实验室

##### 实验：由请求参数控制的用户 ID

看到要求是查看carlos用户的api，登录winenr用户，发现能看到api，就直接修改id发送请求，就查看了

其实这里应该用intruder的orz

#### 使用burp匹配和替换

##### 实验：通过信息泄露绕过身份验证

这里在教程里是先访问了`/admin`页面，但是没说`X-Custom-IP-Authorization`这个是怎么来的，查看了视频之后发现这个通过`trace`请求找到的（不过还是不知道为什么

![](https://pic.imgdb.cn/item/67066421d29ded1a8c9ffb47.png)

在设置中找到http匹配和替换规则，添加一个请求

![](https://pic.imgdb.cn/item/67066484d29ded1a8ca03fde.png)

![](https://pic.imgdb.cn/item/6706659bd29ded1a8ca1065c.png)

现在在网页访问就可以自动添加这个头

### 测试输入验证

---

#### 绕过客户端控件

有些应用依赖于客户端上提交的数据，用户可以完全控制这些数据

##### 实验：对客户端控件的过度信任

在将商品加入购物车之前拦截，然后修改请求体里的数据，然后放行，就可以修改数据，其实感觉和中继器中修改数据差不多（

#### 使用burp测试sql注入漏洞

干扰应用对数据库的查询，就会出现sql注入

可以对想要调查的请求右键>主动扫描，然后查看问题（issues）列表，查看有无sql注入问题

![](https://pic.imgdb.cn/item/6706699ad29ded1a8ca411d0.png)

##### 实验：where子句中的sql注入漏洞

将有sql注入漏洞的请求发送到intruder中，然后负载选择模糊测试-sql注入

还可以在下面的处理中添加一个规则，**Replace placeholder with base**，替换占位符，也就是前面的 **{base}** ，或是用match/replace替换其他占位符

![](https://pic.imgdb.cn/item/67066c12d29ded1a8ca5ae09.png)

然后开始攻击，就可以发现前面长度长的就是成功攻击的

![](https://pic.imgdb.cn/item/67066b6ed29ded1a8ca541e1.png)

#### 跨站点脚本

将恶意的javascript返回给用户，攻击者可以用恶意代码破坏用户和应用的交互

##### 使用burp识别反射输入

将要调查的信息发送到中继器中，然后将参数替换为易于识别唯一的字符串，然后在响应中搜索这唯一的字符串是否在响应中

##### 使用DOM Invader 测试DOM XSS

> DOM XSS:这种攻击是基于浏览器的DOM文档对象，直接在客户端执行恶意脚本，不通过服务器
>
> DOM文档是浏览器将HTML代码转换成的树状结构，允许js动态访问和修改页面内容

启用DOM Invader

在浏览器中的拓展中找到burp后打开

![](https://pic.imgdb.cn/item/6706918dd29ded1a8cc3badd.png)

##### 使用DOM Invader 测试web消息DOM XSS

这个要多启用一个拦截，就是把下面那个也打开

在检查中可以查看到是否访问了信息的`origin`，`data`，`source`

* 如果未访问origin属性，可能是源未经过验证
* 如果未访问data属性，则无法利用该消息
* 如果未访问source属性，可能是源未得到验证

![](https://pic.imgdb.cn/item/670693cad29ded1a8cc58c6e.png)

可以使用下面的bulid poc来生成html来攻击

##### 手动测试反射的XSS

确定XSS注入的位置后，我们注意这里，然后再修改，就可以发现以及成功注入了

![](https://pic.imgdb.cn/item/6706969cd29ded1a8cc7f89c.png)

![](https://pic.imgdb.cn/item/670696ead29ded1a8cc840d8.png)

##### 测试存储的XSS

当应用从不受信任的来源接收数据并以不安全的方式将数据包含在后续的HTTP响应中时，就会出现存储型XSS

首先要测试连接的输入和输出点

在评论中输入test，然后查看含有这个词的http，

![](https://pic.imgdb.cn/item/670699d8d29ded1a8cca960a.png)

将两个请求传输过去，并且新建一个组，两个都添加进去，修改请求，先发送请求，再发送显示页面的请求，就可以查看到存储型XSS了

![](https://pic.imgdb.cn/item/67069a17d29ded1a8ccac4ac.png)

![](https://pic.imgdb.cn/item/67069a3fd29ded1a8ccae27f.png)

##### 通过枚举允许的标签和属性来绕过XSS过滤器

在输入的属性加上<>，之间加上负载，然后查看状态码，200就是成功的

![](https://pic.imgdb.cn/item/67069b9dd29ded1a8ccc679b.png)

就使用这个再次攻击，这些就是允许的属性

![](https://pic.imgdb.cn/item/67069c65d29ded1a8ccd1e13.png)

##### 原型污染

这是一种js漏洞，使攻击者能够将任意属性添加到全局对象原型中，然后用户定义的对象可以继承这些属性，使得攻击者可以控制原本无法访问的对象属性

先打开DOM的原型污染攻击

![](https://pic.imgdb.cn/item/67069e4ad29ded1a8cced6a0.png)

找到有source的界面，然后点击test

![](https://pic.imgdb.cn/item/67069f44d29ded1a8ccfb237.png)

在新页面的console中找到这个property的元素

![](https://pic.imgdb.cn/item/67069fb7d29ded1a8cd018f2.png)

创建一个新对象，并查看到已经被继承

![](https://pic.imgdb.cn/item/6706a03cd29ded1a8cd08cd3.png)

回去选择scan for gadgets，会打开一个页面，扫描得到sink，点击exploit，就会打开一个新页面，并且调用alert（），如果能调用，就有原型污染

![](https://pic.imgdb.cn/item/6706a0b4d29ded1a8cd0fb86.png)

#### OS命令注入

攻击者能在允许应用的服务器上执行任意操作系统（os）命令，由于输入验证不足产生的

##### 测试操作系统命令注入漏洞

在这些易受攻击的网页加入os命令，发现可以执行

![](https://pic.imgdb.cn/item/67078469d29ded1a8c8e883a.png)

##### 异步操作系统命令

这里其实不是很懂，在发送的请求中`nslookup`请求，然后插入一个collaborator payload，发送出去就可以看到

![](https://pic.imgdb.cn/item/670787c6d29ded1a8c9194b8.png)

![](https://pic.imgdb.cn/item/67078818d29ded1a8c91ea40.png)

##### 利用操作系统命令注入漏洞泄露数据

和上一个操作一样，这是在之间加入了一个系统命令whoami，然后在collaborator就能查看到泄露的数据

![](https://pic.imgdb.cn/item/670788fed29ded1a8c930e3a.png)

![](https://pic.imgdb.cn/item/67078922d29ded1a8c93364b.png)

#### XXE注入

干扰应用对XML数据的处理，允许攻击者查看应用服务器文件系统上的文件，并且可以和应用自身可以访问的后端和外部系统进行交互

向应用发送的XML请求中加入有效负载`&xxe`并加入文件路径`<!DOCTYPE foo [<!ENTITY xxe SYSTEM "file:///etc/passwd"> `

![](https://pic.imgdb.cn/item/67078a79d29ded1a8c951f92.png)

##### 盲XXE注入

应用容易收到XXE注入的攻击，但是响应中不返回任意的值，这说明无法查看到服务器端的文件，但是可以用带外技术检测，在XML中加入一个外部实体，然后服务器会向这个实体发送请求，可以通过监视这个请求确定有无

加入的外部实体是通过插入collaborator payload，于是在collaborator 中就能看到请求

![](https://pic.imgdb.cn/item/67078d16d29ded1a8c97dfc6.png)

![](https://pic.imgdb.cn/item/67078d47d29ded1a8c9836b2.png)

#### 测试目录遍历漏洞

要现在过滤中把图片打开，这样才能看到要攻击的参数，然后向参数中加入想要的内容

![](https://pic.imgdb.cn/item/67078ed6d29ded1a8c9a05d8.png)

### 测试点击劫持

诱骗用户点击隐藏的网页元素，攻击者在目标网页上叠加不可见的ui来实现

手动点击非常繁琐，可以用`Clickbandit`工具实现，复制脚本后在浏览器使用

![](https://pic.imgdb.cn/item/6707b5e9d29ded1a8cc15170.png)

![](https://pic.imgdb.cn/item/6707b5e1d29ded1a8cc14a90.png)

记得的要勾选上这个**Disable click actions**选项，这样就不会触发点击事件

![](https://pic.imgdb.cn/item/6707b635d29ded1a8cc18dba.png)

### 测试SSRF漏洞

> **服务器端请求伪造(SSRF)** :允许攻击者诱导服务器端应用向非预期的位置发送请求

首先要查找一个满足两个要求的请求

* 从应用后端获取数据
* 使用用户传输的数据来确定数据从那获取

这里的stockapi中的数据就满足了这个需求

![](https://pic.imgdb.cn/item/6707b805d29ded1a8cc2f85c.png)

通过枚举ip地址获取对这些后端系统的访问权限

![](https://pic.imgdb.cn/item/6707b918d29ded1a8cc3e05f.png)

#### 盲SSRF

攻击者诱导应用向指定的url发送http请求,但是不会返回任何响应

在referer中加入**Collaborator** ,然后发送请求,就可以在**Collaborator** 看到有请求

![](https://pic.imgdb.cn/item/6707ba22d29ded1a8cc4afd8.png)

### 测试WebSocket漏洞

> WebSocket:是支持双向异步通信的长连接,建立连接后会始终保持连接状态,客户端和服务端可以随时随地通信

#### 操作WebSocket信息

找到发送给服务端的信息,发送到中继器,然后修改信息,回到浏览器,会发现成功了

![](https://pic.imgdb.cn/item/6707bd87d29ded1a8cc78c88.png)

![](https://pic.imgdb.cn/item/6707bdb5d29ded1a8cc7ad3e.png)

#### 操作WebSocket握手

点击笔符号可以打开所有WebSocket连接,选择合适的clone,然后就可以修改请求,主机或端口,最后连接,再次进入,可以选择合适的进行连接

![](https://pic.imgdb.cn/item/6707be93d29ded1a8cc84bd2.png)

## 工具

### 仪表盘(dashboard)

用来监视和控制项目中的所有自动化任务

![](https://pic.imgdb.cn/item/6707c08ed29ded1a8cc9c0b7.png)

### burp的浏览器

浏览器中已经配置好了burp的功能,会自动监听,可以拦截

### 代理(proxy)

是浏览器和目标应用之间的Web代理服务器,用来拦截,检查和修改双向传输的流量

#### 拦截

打开拦截后传输的请求就会被卡在这里,不会发送到服务器,可以查看,修改请求,直到放行

![](https://pic.imgdb.cn/item/6707c1d3d29ded1a8ccaca10.png)

#### HTTP历史记录

可以查看HTTP流量记录信息

过滤数据有两个模式,这种就是正常的设置模式

![](https://pic.imgdb.cn/item/6707c344d29ded1a8ccc21f5.png)

还有一种bambda模式,使用java编写

也可以为特殊的请求添加注释或突出显示

#### WebSockets历史

用来查看burp浏览器和web服务器交互的WebSockets信息,可以查看,拦截,修改通信

#### 匹配和替换

可以在设置中添加规则,用来匹配替换,用正则表达式来匹配字符并确定要替换的内容,用来自动替换请求头中的内容

![](https://pic.imgdb.cn/item/6707c4ccd29ded1a8ccd57b3.png)

### 中继器(Repeater)

可以反复修改和发送HTTP或WebSockets信息

#### 分组发送HTTP请求

分组后点击组内的请求发送右边得箭头,可以选择如何发送,一起发送或按顺序发送

![](https://pic.imgdb.cn/item/6707cb15d29ded1a8cd26bbd.png)

### Intruder

对网站执行高度可定制的自动化攻击,可以将不同负载添加到预先定义的位置反复发送请求

#### 攻击类型

* sniper attack
  * 每次将每个有效载荷依次放入每个有效位置,使用一个有效载荷集
* battering ram attack
  * 会同时将有效载荷放入所有有效位置,只使用一个有效载荷集
* pitchfork attack
  * 每个位置迭代不同的有效载荷集,有效载荷同时放入各自的位置,载荷是一一对应的
* cluster bomb attack
  * 每个位置迭代不同的有效载荷集,依次从每个集合中放入,每个载荷都会和另一个载荷集的所有载荷对应

在预定义的载荷集里有些会有占位符,如{base}等,可以在处理中替换掉

#### 资源池

这个是一组共享资源配额的任务,多种攻击时确定资源的使用顺序

#### 经典用途

* 枚举标识符--如密码用户名
* 收集有用的收集
* 模糊测试漏洞--如XSS
* 枚举子域名
* 暴力登录

### 目标(target)

定义工作的范围,查看url视图

站点地图包含以下内容

* 域
* 目录
* 文件
* 参数的请求

### Collaborator

用来诱导应用和Collaborator服务器交互,然后确定交互已发生,用来判断是否有无响应的漏洞

一般是在目标位置插入Collaborator payload,然后在Collaborator页面点击立即轮询按钮,查看交互记数

### DOM Invader

是基于浏览器的工具,用来测试DOM XSS漏洞

### Burp Clickbandit

用来测试点击劫持漏洞

### 比较器

他来比较任意两项数据,用来识别差别

### 解码器

用来进行一些基础的加解密

### 相关工具

右键时会出现的一些工具

#### 分析目标

可以用来分析应用有多少静态和动态url和参数

#### 内容发现

可以抓取可见内容中没查找到的内容和功能

#### 生成CSRF Poc

用来生成CSRF攻击

#### 模拟手动测试

在站点动态上启用

会不定期将常见的测试负载发送到随机的url和测试

#### Organizer

用来存储和注释稍后要返回的HTTP信息副本

### 日志(Logger)

会实时记录生成的所有HTTP流量,可以用Logger来查看,检查拓展的行为

## 自动扫描

右键站点地图或http请求，有三个扫描选项

* 扫描
  * 打开扫描启动器
  * 添加到任务
* 进行被动扫描，分析基本请求和响应的内容，而不是发送自己的请求
* 进行自动扫描，向目标发送请求来探测漏洞

## 设置

burp中有两种设置

* 项目设置，仅适用于当前项目，存储在项目文件本身中
* 用户设置，适用于机器上所有burp项目

## HTTP/2基础知识

### 二进制协议

HTTP/1请求是基于文本的实体（超文本传输协议）因此，服务器用字符串根据分隔符来识别和提取所需的数据

HTTP/2是使用二进制格式，每条信息的开始和结束都有明确的定义，因此不需要分隔符，因此，换行符、冒号等在http/2中没有特殊含义，并且可以以http/1中不能进行的方式注入

### 框架

http2信息以一个或多个单独的帧的形式发送（这个怎么感觉是计算机基础里的http形式，现在应该都是这种了），标头帧相当于http/1请求的请求头和标头，其后跟着就是信息正文数据的数据帧（请求体？）

#### 信息长度

http/2 每一帧前面都有一个明确的长度字段，告诉服务器要读多少字节，服务器可以简单的计算帧长度的总和

### 伪报头

http/2 中，伪报头用来传输有关信息的关键信息（我还以为这个名字是传输假信息的）

总共有五个伪报头

*  **:method** -- 请求的http方法
*  **:path** -- 请求路径，包含查询的字符串
*  **:authority** -- 相当于HTTP/1中的`host`字段，指明要发送到的服务器的主机名和端口号
*  **:scheme** -- 请求方案，通常为http或https
*  **:status** -- 响应状态代码

## burp

在burp中的编辑器会使用http/1的形式显示http/2请求

* 将标题名称中的大写字母转换为小写
* 如果存在`connection`标头会删去
  * 这个一般会长`Connection:Keep-Alive`或`Connection:close`，表示是否保持长连接
* 如果移动了`host`标题，将会返回原始位置
  * 在burp中会先发送伪标头，再发送标头，应该是先发authority，修改在之后

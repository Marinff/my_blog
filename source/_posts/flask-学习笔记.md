---
title: flask学习笔记
date: '2025-05-05 09:37:11'
updated: '2025-05-08 23:52:19'
permalink: /post/flask-study-notes-kl2bh.html
comments: true
toc: true
---



# flask学习笔记

# flask学习笔记

最基础的代码

```python
# 从flask包里导入flask
from flask import Flask

# __name__：代表app.py这个模块
app = Flask(__name__)

# 创建一个路由和视图函数的映射
@app.route('/')

def hello_world():  
    return 'Hello Wor11!'


if __name__ == '__main__':
    app.run()
```

### 一些快捷键

---

alt+enter 快速导入语句所需要的库

ctrl+D 添加一行当前语句

### debug模式

---

默认关闭

![](https://pic.imgdb.cn/item/6694e13dd9c307b7e9375ba1.png)

1.开启debug模式后只要修改代码后保存就行，不用重启                                                 2.出现bug，在浏览器上可以直接看到出错信息

```python
if __name__ == '__main__':
    app.run( debug = ture )
```

或打开flask调试![](https://pic.imgdb.cn/item/6698abb9d9c307b7e94e927e.png)

### 修改host

---

让其他的电脑可以访问到我的flask项目，只要连在同一无线网下，用我的ip地址即可访问

![](https://pic.imgdb.cn/item/66967650d9c307b7e936e9d8.png)

修改额外选项

![](https://pic.imgdb.cn/item/66967695d9c307b7e937813c.png)

### 修改port端口号

---

默认监听5000端口，假设这个被占用，修改port号即可修改端口号

![](https://pic.imgdb.cn/item/669676e2d9c307b7e9384712.png)

## url与视图

---

url：http(https)://www.qq .com:80/path                                                                                                                                      对应       协议              域名              端口号{http[80]   https[443]   这是默认端口}

url与视图：path与视图

```python
@app.route('/')      # '/' 相当于根路由，即path是没有的
              # 前面的app相当于app=Flask(__name__)
```

```python
# 修改path
@app.route('/profile')
def profile():
    return "个人中心"
# 定义多层path
@app.route('/blog/list')
def blog_list():
    return "博客列表"
```

### 定义url参数

---

```python
@app.route('/blog/<int：blog_id>')   # 前面可以修改类型，用<>包裹参数
def blog_detail(blog_id):       # 用()加入参数
	return "您的id是：%s" % blog_id
```

```
string         字符串，接受除/以外的符号
path           路径，类似string，可以接受/
uuid           由一组32位的十六进制所构成
any            指备选值中的任何一个
int float ......
```

将参数固定到了path中，访问时必须要带参数

#### 查询字符串的方法传参

---

/book/list                   ：会返回第一页的数据                                                                                /book/list?page=2    :  会返回第二页的数据

```python
from flask import request
@app.route('/book/list')
def book_list():
    page = request.args.get('page', 1, type=int)
    return f"当前是第{page}页"

# http://127.0.0.1:8000/book/list?page=2
# ('page',1,type=int) 中间的是默认值1
# request.args   args 是arguments(参数)的缩写，这是一个类字典类型，可以用字典的形式获取数据

```

#### 随机取数字

```python
import string

source = string.digits*4 # 乘4代表原始的字符串重复了4次
captcha = random.sample(source,4)
captcha = "-".join(captcha)   # 把列表的分隔符改为-
```

## cookie和session

### cookie

---

用浏览器来储存用户信息的一个文件，储存在电脑本地

存储量很小，一般用来存储登录状态

以键值对进行表示的(key=value)，例如name=marin，则cookie的名字是name，信息的marin

cookie分为对话性存储和持久性存储，会话性只会保存在客户端的内存，关闭客户端就失效，持久性会保存在硬盘中，直到生存期结束或者用户销毁

#### flask实现cookie

---

```python
# 创建cookie

from flask import Flask, render_template, make_response
from datetime import datetime

@app.route('/cookie')
def cookie():
    response = make_response( "设置cookie成功")
    expires_time = datetime.datetime.now() + datetime.timedelta(days=1)
    response.set_cookie('name',"marin",expires=expires_time) 
    # max_age=10 最大存在时间，单位是s
    # expires = expires_time 存在的时间戳
    
    return response

--------------------------------------------------------
# 获取cookie

from flask import request


@app.route('/get_cookie')
def get_cookie():
    cookies =  request.cookies # 一下获取所有
  #  cookies = request.cookies.get("name")  获取一个cookie
    cookies_dict = request.cookies.to_dict()
    print(cookies)
    print(cookies_dict)
# ImmutableMultiDict([('name','marin'), ('sex','nan')])
# {'name': 'marin', 'sex': 'nan'}


    return "1"
--------------------------------------------------------
# 删除cookie

@app.route('/del_cookie')
def del_cookie():
    response = make_response("删除cookie成功")
    response.delete_cookie("name")
    return response
```

### session

---

被称为会话控制，用来存储特定用户会话所需的属性及配置，保存在服务端

我们使用的互联网应用层协议基本基于http和https，这两个是无状态的，只负责请求和响应，所以没有额外操作，服务器不知道你是谁，无法给你展示与你相关的内容

所以session和cookie是一般一起用

#### 工作原理

---

1.用户第一次请求服务器，服务器生成一个sessionid

2.服务器将生成的sessionid返回给客户端

3.客户端收到sessionid会保存在cookie中，当客户再次访问服务端时会带上这个sessionid

4.当服务端再次接受到来着客户端的请求时，会检查是否存在sessionid，    不存在就新建一个sessionid，重复1，2的流程                                                                   存在就查询服务端的session，找到与这个sessionid相对应的文件，文件中的键值就是sessionid，值为用户的信息

session通常存储在服务端的内存或是redis数据库中，也可以存储在硬盘文件或者mysql中

#### flask实现session

---

```python
# 启用session
app.config['SECRET_KEY'] = os.urandom(24)  
# 生成随机数种子，用于生成sessionid
--------------------------------------------------------
# 添加session
from flask import session

@app.route("/add_session")
def add_session():
    session['username'] = "marin"
    session['role'] = "root"
    return 'session added'
--------------------------------------------------------
# 获取session
@app.route("/get_session")
def get_session():
    print(session)
    username = session.get('username')
    return "session get"+"username:"+username
--------------------------------------------------------
# 删除session
@app.route("/del_session")
def del_session():
    session.clear()
    return 'session deleted'
```

## 拦截器

---

实现项目开发的权限处理

### 全局拦截器

---

对所有请求进行拦截

```python
from flask import Flask,request
@app.route('/login')
def login():
    session["is_login"] = True
    return "登录成功"

@app.route("/updata_user")
def update_user():
    return "修改成功"

@app.before_request
def before_request():
    # 获取用户的url，根据url进行判断是否拦截
    url = request.path
    if url == '/login':
        pass
    else:
        # 利用session来判断是否成功登录
        if_login = session.get('is_login')
        if if_login != True:
            return " 请登录"

```

### 模块拦截器

---

对当前模块的请求进行拦截

```python
@User.before_request
def before():
    if request.path.startswith('/v'):
        pass
    else:
        return "请登录"


@User.route('/v/user/login')
def login():
    return "is login"


@User.route('/v/user/register')
def register():
    return "is register"

@User.route('/user/userinfo')
def userinfo():
    return "is user"
```

### 黑白名单

---

```python
@app.before_request
def before_request():

    url = request.path
    pass_path = ['/','login','register']
    suffix = url.endswith("png") or url.endswith("jpg") or url.endswith("css")
    if url in pass_path or suffix:
        pass
```

### 错误处理

---

```python
@app.errorhandler(404)
def not_found(error):
    print(error)
    return "输入的网址错误"

@app.errorhandler(500)
def server_error(error):
    return "服务端出错"
```

## 模板渲染

---

在templates文件夹新建html文件,这里好像涉及到了一个新东西：jinja2模板

### bootstrap

---

好像是美化模板，就先没学orz

### jinja2

---

是一个flask默认的模板引擎

#### 基本使用

---

在templates文件夹中新建html文件

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
</html>

# 这是默认的html模板
# 在body中可以加入东西，title可以修改网页标题
```

```python
from flask import render_template

@app.route('/')
def hello_world():
    return render_template("index.html")   
# render_template 会默认从当前项目的文件夹下寻找文件，并解析渲染

```

#### 变量传递

---

```python

@app.route('/blog/<blog_id>')
def blog(blog_id):
    return render_template("blog_detail.html",blog_id=blog_id,usrname="marin")
--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>博客详情</title>
</head>
<body>
<h1>您的用户名是：{{ usrname }}</h1>
<h1>你访问的博客id是：{{ blog_id }}</h1>
</body>
</html>

# 参数用{{  }}来包裹，依赖于jinja2模板
```

```python
# session参数的传递
@dp.route("/index")
def index():
    session["username"]= "marin"
    return render_template("index.html")
--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>marin_blog</title>
</head>
<body>
<h1>blog首页</h1>
<div>{{ session.get("username") }}</div>

</body>
</html>
--------------------------------------------------------
# 类
class usr:
    def __init__(self,name,email):
        self.name = name
        self.email = email

@app.route('/')
def hello_world():
    user1 = user("xxx","1@qq.com")
    return render_template("index.html",user1=user1)
--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>marin_blog</title>
</head>
<body>
<h1>blog首页</h1>
<h2>{{ user1.name }} / {{ user1.email }}</h2>
</body>
</html>

#  用打点调用来读取数据
--------------------------------------------------------
# 字典
@app.route('/')
def hello_world():
    user1 = user("xxx","1@qq.com")
    person = {
        "username":"marin",
        "email":"1.com"
    }
    return render_template("index.html",user1=user1,person=person)
--------------------------------------------------------

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>marin_blog</title>
</head>
<body>
<h1>blog首页</h1>
<h2>{{ user1.name }} / {{ user1.email }}</h2>
<div>{{ person['username'] }} / {{ person.email }}</div>
</body>
</html>

# 字典可以用[]调用，也可以用打点调用
```

#### 蓝图下的使用

---

```python
app = Flask(__name__,template_folder='templates')
```

#### 过滤器

---

通过"|"实现

```python
@app.route('/filter')
def filter():
    user2 = user("xxxx","1@qq.com")
    return render_template("filter.html",user2=user2)
--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{{ user2.name }}-{{ user2.name|length }}
</body>
</html>

# "|" 是用来过滤的，后面是过滤的要求，length是求长度
```

下面是找到的内置过滤器（

先记着orz

**​`safe`​**: (常用)标记一个变量为安全的，即在输出时不进行 HTML 转义。

```
 {{ "<script>alert('XSS');</script>"|safe }}
```

**​`capitalize`​**: 将字符串的第一个字母大写，其余字母小写。

```
 {{ "hello world"|capitalize }}
```

**​`lower`​**: 将字符串转换为小写。

```
 {{ "HELLO WORLD"|lower }}
```

**​`upper`​**: 将字符串转换为大写。

```
{{ "hello world"|upper }}
```

**​`title`​**: 将字符串中的每个单词的首字母大写。

```
 {{ "hello world"|title }}
```

**​`trim`​**: 删除字符串两端的空白字符。

```
 {{ "   hello world   "|trim }}
```

**​`striptags`​**: 移除字符串中的所有 HTML/XML 标签。

```
{{ "<p>Hello World!</p>"|striptags }}
```

**​`urlize`​**: 将 URL 转换为 HTML 链接。

```
{{ "Visit http://www.example.com"|urlize }}
```

**​`urlencode`​**: 对 URL 进行 URL 编码。

```
 {{ "http://www.example.com?q=hello"|urlencode }}
```

**​`date`​**: 格式化日期时间。

```
 {{ "2023-01-01 12:00:00"|datetime("%Y-%m-%d") }}
```

**​`filesizeformat`​**: 将字节大小转换为人类可读的格式。

```
{{ 1024*1024*1024|filesizeformat }}
```

**​`join`​**: 将序列中的元素连接成一个字符串。

```
 {{ ["apple", "banana", "cherry"]|join(", ") }}
```

**​`length`​**: 返回序列或字符串的长度。

```
{{ "hello"|length }}
```

**​`replace`​**: 替换字符串中的子串。

```
{{ "hello world"|replace("world", "there") }}
```

**​`reverse`​**: 反转字符串或序列。

```
{{ "hello"|reverse }}
```

**​`slice`​**: 截取序列的一个片段。

```
{{ [1, 2, 3, 4, 5]|slice(1, 3) }}
```

接下来是自定义过滤器

```python
def data_format(value,format = "%Y年%m月%d日 %H:%M:%S"):
    return value.strftime(format)

# 定义一个函数 value是导入的值 将value传入的值按照format的形式改变后返回给value

app.add_template_filter(data_format,"d_format")

# 将函数data_format添加到flask的过滤器里，名字是d_format，可以用这个名字来调用这个过滤器

@app.route('/filter')
def filter():
    mytime = datetime.now()
    return render_template("filter.html",mytime=mytime)

```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{{ mytime | d_format }}
</body>
</html>
```

#### 控制语句

---

```python
# if语句
@app.route('/control')
def control():
    age  = 19
    return render_template("filter.html",age=age)

--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{% if age > 18 %}
    <div>已成年</div>
{% elif age == 18 %}
    <div>刚满18岁</div>
{% else  %}
    <div>未满18岁</div>
{% endif %}
</body>
</html>

# 要有end if 结尾

--------------------------------------------------------
# for语句
@app.route('/control')
def control():
    books = [{
        "name":"1",
        "author":"11"
    },
        {
         "name":"2",
         "author":"22"
        }]
    return render_template("filter.html",books=books)
--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{% for book in books %}
<div> {{ book.name }}  {{ book.author }}    </div>
{% endfor %}

</body>
</html>

# 不存在break语句，循环只能一次循环完


# 用{%   %}包裹控制语句
# jinja2不需要缩进，只是为了方便阅读
```

#### 全局函数

---

在jinja2中调用python的函数

```python
# 在app.py中实现

def add(a,b):
    return a+b
app.jinja_env.globals.update(myadd = add) # 全局函数设置
--------------------------------------------------------
<div>{{ myadd(1,2) }}</div>
```

利用蓝图上下文实现

```python
@dp.context_processor
def my_func2():
    def add(a,b):
        return a+b

    return dict(my_add=add)
--------------------------------------------------------
<div>{{ my_add(4,5) }}</div>


--------------------------------------------------------
# 法2
@dp.context_processor
def my_func():
    def add(a,b):
        return a+b
    return { "my_add2": add}
--------------------------------------------------------
<div>{{ my_add2(4,5) }}</div>
```

#### 测试器

---

利用if is语句进行的操作

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
{% if user is defined %}
    user定义了
{% else %}
	user没被定义
{% endfor %}

</body>
</html>

# is后面的"defined"就是测试器
```

![](https://pic.imgdb.cn/item/66989868d9c307b7e939ec64.jpg)

#### 表格语句

---

-  **​`<table>`​** : 定义一个表格的开始和结束。
-  **​`<tr>`​** : 定义表格中的一行。
-  **​`<td>`​** : 定义表格中的一个标准单元格。
-  **​`<th>`​** : 定义表格中的标题单元格，通常用于表格头部，文本默认居中并加粗。
-  **​`<thead>`​** : 包含表格的头部信息，即列标题。
-  **​`<tbody>`​** : 包含表格的主体部分，即数据行。
-  **​`<tfoot>`​** : 包含表格的底部信息，如总计或脚注。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<table>
<thead>书单</thead>

<tr>                       # 定义表格的一行
<th>序号</th>				 # 定义这行的一格，加粗
<th>书名</th>
<th>作者</th>
<td>备注</td>				# 定义一格 ，不加粗
</tr>

<tbody>

{% for book in books %}
    <tr>           # <tr>放在for循环里是为了每次都创建新的一行
    <td>{{ book.list }}</td>
    <td>{{ book.name }}</td>
    <td>{{ book.author }}</td>
    </tr>
{% endfor %}

</tbody>
</table>

</body>
</html>
```

![](https://pic.imgdb.cn/item/66987952d9c307b7e91a95be.png)

#### 元素的样式

---

- `color`: 设置文本颜色。
- `background-color`: 设置背景颜色。
- `font-size`: 设置字体大小。
- `font-family`: 设置字体系列。
- `padding`: 设置内边距。
- `margin`: 设置外边距。
- `border`: 设置边框。
- `width` 和 `height`: 设置宽度和高度。
- `text-align`: 设置文本对齐方式。
- `display`: 设置元素的显示类型，如 `block`, `inline`, `flex`, `grid` 等。
- `position`: 设置定位类型，如 `static`, `relative`, `absolute`, `fixed` 等。
- `float`: 设置浮动。
- `clear`: 设置清除浮动。
- `overflow`: 设置溢出内容的显示方式。
- `visibility`: 设置元素的可见性。

<div>
<div style="color: black; background-color: green;">   这段文字就是下面代码的效果
</div>

```python
<div style="color: black; background-color: green;">   这段文字就是下面代码的效果
</div>
```

这个好像也可以通过css文件来实现，还没学到（

#### 模板继承

---

网页的大部分模块是重复的 ，把这些模块的东西写入父模板，子模板直接继承就行了

```html
# 父模板
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}{% endblock %}</title>
</head>
<body>

<ul>
    <li><a href="#"> 首页 </a> </li>
    <li><a href="#"> 新闻 </a> </li>
</ul>

父模板
{% block body %}
{% endblock %}

<footer>底部标签</footer>

</body>
</html>

# 里面<ul><li>是啥还没学，先不管，父模板中固定的内容会直接给到子模板，而子模板修改的是里面{% block %} 的内容

--------------------------------------------------------
#子模板

{% extends "base.html" %}    # 导入父模板的内容

{% block title %}         # 修改父模板中的block
	子模板 				#写一个block之后按tab
{% endblock %}

{% block body %}
	字模板内容
{% endblock %}



```

#### 加载静态文件

---

这就用到上面的css，图片等文件

```html
# 图片
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

<img src="{{ url_for('static',filename='marin.jpg') }}" alt="">

# img之后按tab键就会出现，url_for是调用的函数'static'是默认要有的，filename是要文件的路径，如果就在static文件夹之下就不用路径

</body>
</html>

--------------------------------------------------------

```

```html
# css文件
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    
    
    <link rel="stylesheet" href="{{ url_for('static',filename='css/style.css') }}">

# 加载css文件，用link再按tab得到这段，还是用url_for    
    
</head>
<body>
</body>
</html>
--------------------------------------------------------
# css文件
body{
    background-color:pink;
}
```

```html
# js文件
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
    
    <script src='{{ url_for('static',filename="js/my.js") }}'></script>
    
# 输入script再按tab ，在前面加src={{}}，在用url_for函数   
    
</head>
<body>

</body>
</html>
--------------------------------------------------------
alert("my.js中进行的")
```

## 数据库（法一

---

安装了sql，pymysql，flask-sqlalchemy

```bash
pip install pymysql
pip install flask-sqlalchemy
pip install sqlalchemy
```

接着下载了个navicat连接sql，跟着下面的博客下载的

[超详细工具Navicat安装教程-CSDN博客](https://blog.csdn.net/mouserat11/article/details/137273520)

创建一个数据库

![](https://pic.imgdb.cn/item/6698c8dfd9c307b7e973faa5.png)

下面是连接数据库的代码，不知道原理orz

```python

from flask_sqlalchemy import SQLAlchemy

# mysql所在的主机名
HOSTNAME = "127.0.0.1"
# mysql监听的端口号，默认3306
PORT = 3306
# 连接mysql的用户名
USERNAME = "root"
# 连接mysql的密码
PASSWORD = "marin"
# mysql上创建的数据库名称
DATABASE = "database_learn"

app.config['SQLALCHEMY_DATABASE_URI'] = f"mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}/{DATABASE}?charset=utf8mb4"

# 在app.config中设置好连接数据库的信息
# 然后使用SQLALchemy（app）创建一个db对象
# SQLALchemy会自动读取app.config中连接数据库的信息

db = SQLAlchemy(app)
```

---

下面是ai对app.config那句代码的解析，感觉会忘，记着先

这行代码是在 Flask 应用中配置数据库连接 URI 的关键部分。它使用 Python 的 f-string（格式化字符串字面量）来构建一个符合 SQLAlchemy 要求的数据库连接字符串。让我们详细解析一下：

- `app.config['SQLALCHEMY_DATABASE_URI']`：这是 Flask 应用配置字典中的一个键值对，`SQLALCHEMY_DATABASE_URI` 是一个特殊的键，用于告诉 SQLAlchemy 如何连接到数据库。你将一个字符串赋值给这个键，这个字符串包含了连接到数据库所需的所有信息。
- `f"mysql+pymysql://"`：这部分指定了数据库的类型和驱动程序。`mysql` 表示你要连接的是 MySQL 数据库，`+pymysql` 指定了使用 pymysql 作为 Python 的 MySQL 驱动程序。
- `${USERNAME}:${PASSWORD}@`：这部分包含了数据库的用户名和密码。`${USERNAME}` 和 `${PASSWORD}` 分别被 f-string 解析为你之前定义的 `USERNAME` 和 `PASSWORD` 变量的值。
- `${HOSTNAME}/`：这部分指定了数据库服务器的主机名或 IP 地址。`${HOSTNAME}` 同样会被解析为之前定义的 `HOSTNAME` 变量的值。
- `${DATABASE}?`：这部分指定了具体的数据库名称。`${DATABASE}` 会被解析为之前定义的 `DATABASE` 变量的值。
- `charset=utf8mb4`：最后，`charset=utf8mb4` 指定了连接时使用的字符集。`utf8mb4` 是一个支持四个字节 Unicode 字符的 UTF-8 兼容字符集，这意味着它可以存储全世界几乎所有的字符，包括表情符号等。

综上所述，这行代码构建了一个完整的数据库连接字符串，告诉 SQLAlchemy 如何连接到你的 MySQL 数据库，使用什么用户名、密码、主机名、端口、数据库名称以及字符集。一旦这个配置设置完成，SQLAlchemy 就可以使用这个信息来建立与数据库的连接。

---

```python
from sqlalchemy import text

with app.app_context():
    with db.engine.connect() as conn:
        rs = conn.execute(text("select 1"))
        print(rs.fetchone())    # 输出1就连接成功
# 这是基于上面连接的数据库是否成功
```

另一个连接数据库的方法

```python
import pymysql

# 建立与数据库的连接
conn = pymysql.connect(
    host = '127.0.0.1',
    port = 3306,
    user = "root",
    password = "marin",
    database = "test",
    charset = "utf8mb4",
    cursorclass = pymysql.cursors.DictCursor
# 改变游标的返回格式，从元组改为字典
)

# 执行sql语句
sql = "select * from users"	# 查询users表中的所有记录
# 实例化一个游标对象
cursor = conn.cursor()	
# 创建游标对象，用于执行SQL语句和获取结果
cursor.execute(sql) 	 # 通过游标执行SQL语句


# 获取查询之后的所有结果
result  = cursor.fetchall()
print(result)

# 关闭连接
cursor.close()		# 关闭游标
conn.close()	    # 关闭数据库连接
```

### ORM模型与表的映射

---

ORM中文是 对象关系映射，一个ORM模型和数据库中的一个表对应

```python
class User(db.Model):   # 继承模型，成为orm
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True,autoincrement = True)  
    # integer是整形,最后是自动增长的意思
    # string是变动字形，括号里是长度
    username = db.Column(db.String(100), nullable=False)
    password = db.Column(db.String(100), nullable=False)

    
    
user =User(username='marin', password='admin')
# sql: insert user(username,password) values('marin','admin');



with app.app_context():      # 这个是请求上下文，还没学到 *
    db.create_all()          # 在数据库里创建这个表
```

![](https://pic.imgdb.cn/item/6699c113d9c307b7e975e903.png)

### ORM模型的操作

---

```python
# 添加数据到数据库中

@app.route('/user/add')
def add_user():
    # 1.创建orm对象
    uesr = User(username='marin', password='1111111')
    # 2.将orm对象添加到db.session
    db.session.add(uesr)
    # 3.将db.session中的改变同步到数据库中
    db.session.commit()
    return 'User added!'



--------------------------------------------------------
# 查找数据

@app.route('/user/query')
def query_user():
    # 1.get查找：根据主键查找 ,query是db.model提供的
    user = User.query.get(1)
    print(f"{user.id}-{user.username}-{user.password}")
    
    
    # 2.filter_by查找
    # Query 是一个类数组的对象
    users = User.query.filter_by(username = 'marin')
    for user in users:
        print(user)
    
    return "数据查找成功"

--------------------------------------------------------
# 修改数据

@app.route('/user/update')
def update_user():
    # first返回user对象，不是query对象
    user = User.query.filter_by(username='marin').first()
    user.password = '222222'
    db.session.commit()
    return "数据修改成功"

--------------------------------------------------------
# 删除数据

@app.route('/user/delete')
def delete_user():

    user = User.query.get(1)
    # 从db.session中删除
    db.session.delete(user)
    # 将删除同步到数据库中
    db.session.commit()
    return  "数据删除成功"
```

### 表关系

---

#### 外键

---

```python
class Article(db.Model):
    __tablename__ = 'article'
    id = db.Column(db.Integer, primary_key=True,autoincrement = True)
    title = db.Column(db.String(200), nullable=False)
    content = db.Column(db.Text, nullable=False)
    # 因为string字符只能储存255个字符，所以改为text
    # 添加外键，引用的是user的外键
    author_id =                                db.Column(db.Integer, db.ForeignKey('user.id'))
    author = db.relationship("User",backref='articles')
    # 使用user的id，去user里面查找数据，再传给author,给
	# backref自动给User自动添加articles的属性
    
 ------------------------------------------------------
	author = db.relationship("User",back_populates="articles")  # 反向引用，在article中返回articles，但是要在user中加上
    articles = db.relationship("Article",back_populates="author")
    
    # 这个方法代码看起来明显一点
-------------------------------------------------------    
    
    
article = Article(title="flask",content = "flaskxxxxx")
article.author_id = user.id  # 引用user的id
print(article.author) # 找到了user的数据

--------------------------------------------------------

@app.route("/article/add")
def add_article():
    article1 = 
    Article(title="flask", content="flaskxxxxxxx")
    article1.author = User.query.get(2)

    article2 = Article(title="django", content="xxxxxxx")
    article2.author = User.query.get(2)

    # 添加到session
    db.session.add_all([article1, article2])
    db.session.commit()

    return "文章添加成功"

@app.route("/article/query")
def query_article():
    user = User.query.get(2)
    for article in user.articles:
        print(article)
    return "文章查找成功"

```

![](https://pic.imgdb.cn/item/669e0896d9c307b7e9ab854f.png)

#### 单表关系

---

```python
# 或  or_()
# 和  and_()
result = db_session.query(User).filter(or_(User.user_id==1,User.user_id==2)).all()

result = db_session.query(User).limit(4).all()

result = db_session.query(User).limit(4).offset(2).all()

result = db_session.query(User).filter(User.user_id < 10).all()

# order by    排序
result = db_session.query(User).order_by(User.user_id.desc()).all()

# like 模糊查询，查询带有%%之间的
result = db_session.query(User).filter(User.nickname.like('%2%')).all()

# sum  使用函数
result = db_session.query(func.sum(User.user_id)).all()
```

#### 双表关系

---

```python
class User(Base):
    # 表结构的反射加载
    __table__ = Table("user", Base.metadata,autoload_with=engine)

class Article(Base):
    __table__ = Table("article", Base.metadata,autoload_with=engine)

@app.route("/")
def index():
    username=request.args.get("username")
    # all_article = (
    #     db_session.query(User,Article).join(Article,User.user_id == Article.user_id).filter(User.username==Article.username).all())
    all_article = (
        db_session.query(User.username,User.nickname,Article.article).join(Article,User.user_id == Article.user_id).filter(User.username==Article.username).all())


    for u,n,a in all_article:
        print(u)
        print(n)
        print(a)

    return "qsdz"
```

#### 多表关系

---

```python
@app.route("/")
def index():
    username=request.args.get("username")
    all_article = db_session.query(User,Article,Favour).outerjoin(
    Article,User.user_id == Article.user_id).outerjoin(
        Favour,Article.id==Favour.article_id).filter(User.username == username).all()

    print(all_article)
    for u,a,f in all_article:
        print(a.id)
        print(u.nickname)
        print(f.id)

    return "qsdz"
```

#### 数据返回

---

```python
@app.route("/")
def index():
    keyword=request.args.get("keyword")
    result = db_session.query(Article).filter(or_(Article.title.like("%"+ keyword +"%"),Article.user_id.like("%"+keyword+"%"))).all()

    print(result)
    to_page_data = model_list(result)

    return to_page_data

def model_list(result):
    list_result=[]
    for row in result:
        print(row.__dict__)
        my_dict = {}
        for k,v in row.__dict__.items():
            if not k.startswith("_sa"):
                my_dict[k] = v
        print(my_dict)
        list_result.append(my_dict)
    print(list_result)
    return list_result
```

### migrat迁移ORM数据库

---

```python
with app.app_context():
    db.create_all()  
# 不会识别到新增的字段，假设user多了一个数据，不会映射到数据库    --------------------------------------------------------
pip install flask-migrate  下载
--------------------------------------------------------
from flask_migrate import Migrate

migrate = Migrate(app, db)

# ORM 模型映射三步 
# 都是在终端执行的命令
# flask db init ：只需要执行一次
# flask db migrate：识别ORM模型的改变，生成迁移脚本
# flask db upgrade： 运行迁移脚本，同步到数据库中
# 每次修改ORM的时候就可以执行后面两步，就可以自动同步
```

### 数据库（法二

---

```python

from flask import Flask
from sqlalchemy import create_engine

app = Flask(__name__)

# 创建一个引擎，连接到数据库
engine = create_engine('mysql+pymysql://root:marin@127.0.0.1:3306/practice')
```

一个创建数据库的方法，不常用

```python
from flask import Flask
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import sessionmaker, scoped_session, declarative_base

app = Flask(__name__)

# 创建一个引擎，连接到数据库
engine = create_engine('mysql+pymysql://root:marin@127.0.0.1:3306/practice')
# 打开数据库的连接会话
session = sessionmaker(engine)
# 保证线程安全
db_session = scoped_session(session)
# 获取基类
Base = declarative_base()

class User(Base):
    __tablename__ = 'user'
    user_id = Column(Integer, primary_key=True)
    username = Column(String(20), unique=True)

User.metadata.create_all(engine)
```

获取数据库的数据

```python
from flask import Flask
from sqlalchemy import create_engine, Table
from sqlalchemy.orm import sessionmaker, scoped_session, declarative_base

class User(Base):
    # 表结构的反射加载
    __table__ = Table("user", Base.metadata,autoload_with=engine)


@app.route('/',methods = ['post'])
def hello_world():
    result = db_session.query(User).all()
    print(result)
    for r in result:
        print(r.username)
    return '登录成功'
```

数据查询，返回的数据查看

```python
@app.route('/',methods = ['post'])
def hello_world():
    # result = db_session.query(User).all()
    # print(result)
    # for r in result:
    #     print(r.username)
    requst_data = request.data
    requst_data = json.loads(requst_data)
    user_input_name = requst_data['username']
    print(user_input_name)
    print(requst_data)

    # filter方法，写的不是之前的参数，支持 == > < <= >=
    result = db_session.query(User).filter(User.username == user_input_name).first()

    # 在正确查询数据库数据正确，需要做一个密码的md5转换
    # user_input_password = hashlib.md5(user_input_name.encode()).hexdigest()

    # 用之前的运算符 =
    result = db_session.query(User).filter_by(username = user_input_name).first()
    print(result.username)

    return '登录成功'

```

```python
@app.route('/',methods = ['post'])
def reg():

    requst_data = request.data
    requst_data = json.loads(requst_data)
    username = requst_data['username']
    password = requst_data['password']
    nickname = requst_data['nickname']
    picture = requst_data['picture']

    insert_data = {
        "username": username,
        "password": password,
        "nickname": nickname,
        "picture": picture
    }

    user = User(**insert_data)
    db_session.add(user)
    db_session.commit()

    return '注册成功'
# 数据更新
# 查询要改动过的行，再删除或是更新
row = db_session.query(User).filter_by(user_id = 11).first()
row.nickname = "aaaa"
db_session.commit()

# 数据删除
row = db_session.query(User).filter_by(user_id = 11).delete()
db_session.commit()
```

## Blueprint

---

为了好管理，所以会把不同的识图给模块化，不会全塞在app.py中，blueprint就是用来模块化的，blueprint相当于flask的子模块

![](https://pic.imgdb.cn/item/669e1e1cd9c307b7e9c0ddfa.png)

```python
# user.py模块

from flask import Blueprint

# 在这个蓝图中的识图函数都要从/user中
bp = Blueprint("user",__name__,url_prefix='/user')

# user/login
@bp.route("/login")
def login():
    pass

--------------------------------------------------------
#导入，在app.py文件中

from blueprint.qa import bp as qa_bp
from blueprint.user import bp as user_bp

# 绑定
app.redirect_blueprint(qa_bp)
app.register_blueprint(user_bp)
```

实际项目中配置不会放在app.py中，而是在额外的模块中实现后导入，其他的模块要用就直接导入这个配置文件，为了解决循环引用的问题

```python
# 例子
--------------------------------------------------------
# app.py

from flask import Flask
import config                  
from exts import db			 # 导入db
from models import User		# 导入models中的模块
--------------------------------------------------------
# models.py

from exts import db        # 也要导入db

class User(db.Model):    # 用db来新建一个user
    pass
--------------------------------------------------------
# exts.py
# 就是为了解决循环引用
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
--------------------------------------------------------
```

```python
from flask import Flask
import config     # 在项目中新建一个config.py文件
from exts import db


app = Flask(__name__)
# 绑定配置文件
app.config.from_object(config)

db.init_app(app)  
# 可以用这个语法绑定db到app，因为在exts中db并没有绑定app
```

### 上下文处理器

---

```python
@dp.context_processor
def my_func():
    # 返回值必须是字典类型
    return {"title" : "1111"}
# 上下文里的内容所有都可以使用

@dp.context_processor
def my_func1():
    my_dict = {"key1":"value1","key2":"value2"}
    return my_dict

@dp.route("/index")
def index():
    return render_template("index.html")

--------------------------------------------------------
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>marin_blog</title>
</head>
<body>

<h1>blog首页</h1>
<div>{{ title }}</div>

</body>
</html>


--------------------------------------------------------
@dp.context_processor
def my_func1():
    my_dict = {"key1":"value1","key2":"value2"}
    return dict(my=my_dict)
# 这个方法对应的html语法
<div>{{ my.key1 }}</div>
```

### 实践1

---

```python
# config.py

HOSTNAME = '127.0.0.1'
PORT = 3306
USERNAME = 'root'
PASSWORD = 'marin'
DATABASE = "practice"


DB_URI = f"mysql+pymysql://{USERNAME}:{PASSWORD}@{HOSTNAME}:{PORT}/{DATABASE}?charset=utf8mb4"
SQLALCHEMY_DATABASE_URI = DB_URI

```

```python
# model.py

from exts import db
from datetime import datetime

class User(db.Model):
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True,autoincrement=True)
    username = db.Column(db.String(100),nullable=False)
    password = db.Column(db.String(100),nullable=False)
    email = db.Column(db.String(100),nullable=False,unique=True)
    join_time = db.Column(db.DateTime,default=datetime.now)
    # unique 邮箱不能相同
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>注册</title>
</head>
<body>
<form action="{{ url_for('user.register') }}" method="POST">
--------------------------------------------------------
    ## 如果你的应用中使用了蓝图（Blueprints），那么在引用终点时需要加上蓝图的名字，例如 user.register。 
    一直在报错，就是一开始只写了register
-------------------------------------------------------    
    <table>
        <tr>
            <td>用户名：</td>
            <td><input type="text" name="username"></td>
        </tr>
        <tr>
            <td>邮箱：</td>
            <td><input type="email" name="email"></td>
        </tr>
        <tr>
            <td>密码：</td>
            <td><input type="password" name="password"></td>
        </tr>
        <tr>
            <td>确认密码：</td>
            <td><input type="password" name="cofirm_password"></td>
        </tr>
        <tr>
            <td></td>
            <td><input type="submit" value="提交"></td>
        </tr>
    </table>
</form>
</body>
</html>
```

#### 注册表单

---

```
pip install flask-wtf
```

```python
import wtforms
from wtforms.validators import Email,Length,EqualTo
# EqualTo 判断两个是否相同
from models import User

# 验证数据是否符合要求
class RegisterForm(wtforms.Form):
    email = wtforms.StringField(validators=[Email(message="邮箱格式错误")])
    username = wtforms.StringField(validators=[Length(min=1, max=20)])
    password  = wtforms.StringField(validators=[Length(min=3,max=20,message="密码格式错误")])
    confim_password = wtforms.StringField(validators=[EqualTo("password")])

    # 用来判断是否满足条件
    def validate_email(self,field):
        email = field.data
        user = User.query.filter_by(email=email).first()
        if user:
            raise wtforms.ValidationError(message="邮箱已被注册")

    def validate_username(self,field):
        username = field.data
        user = User.query.filter_by(username=username).first()
        if user:
            raise wtforms.ValidationError(message="用户名已被注册")

```

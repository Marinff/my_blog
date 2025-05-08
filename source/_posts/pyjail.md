---
title: pyjail
date: '2025-05-05 09:37:11'
updated: '2025-05-08 23:53:08'
permalink: /post/pyjail-zhnwjt.html
comments: true
toc: true
---



# pyjail

## Pyjail   ——   沙箱逃逸

pyjail主要是交互式地使用`eval`或者`exec`执行python代码，执行的代码和上下文都会限制

### 前置知识点

#### python类的继承

在python中创建一个类，都会继承一个基类，如果没有指定，就会默认继承`object`，就会继承object的属性和方法

```py
class A(object):
    name = "A"
```

默认继承的方法

```
'__class__'
'__delattr__'
'__dict__'
'__dir__'
'__doc__'
'__eq__'
'__format__'
'__ge__'
'__getattribute__'
'__getstate__'
'__gt__'
'__hash__'
'__init__'
'__init_subclass__', 
'__le__'
'__lt__'
'__module__'
'__ne__'
'__new__'
'__reduce__'
'__reduce_ex__'
'__repr__'
'__setattr__'
'__sizeof__'
'__str__'
'__subclasshook__'
'__weakref__'
'name'
```

如果觉得父类的方法写的不好，可以重写

#### 常用的魔术方法

魔术方法是python中的一种高级语法，允许在类中自定义函数，并绑定到类的特殊方法中

##### 内置的方法

1. `__init__`：用于python类的构造，对类进行初始化操作，在实例化一个类的时候会调用

   ```py
   class A(object):
       def __init__(self):
           print('success')

   # 在这个时候会输出success
   a = A()
   ```
2. `__str__`：返回对象的名称和地址，有几种调用方式
   会在`print(a)`时被调用
   在`str(a)`时被调用，这个其实就是调用了`a.__str__()`来实现

   ```py
   class A(object):
       def __str__(self):
           return "success"
   a = A()
   # 就会输出success
   print(a)
   print(str(a))
   print(a.__str__())
   ----------------------------------------------------------
   class A(object):
       pass
   a = A()
   # 输出 <__main__.A object at 0x0000021BB15C9950>
   print(a)
   ```
3. 比较操作
   `__eq__`：相等比较，`==`
   `__ne__`：不相等比较，`!=`
   `__lt__`：小于比较，`<`
   `__gt__`：大于比较，`>`
   `__le__`：小于或等于比较，`<=`
   `__ge__`：大于或等于比较，`>=`
4. `__delattr__`：删除对象的属性
   在`del a.stu`，就会调用`a.__delattr__(stu)`
5. `__subclasses__`：返回当前类的所有子类，虽然不是继承自object的，但是可以在任何类上调用

   ```py
   class A(object):
       pass

   class B(A):
       pass
   class C(A):
       pass
   # 最后输出的是[__main__.B, __main__.C]
   A.__subclasses__()

   ```
6. `__dict__`：查看内部属性名和属性值组成的字典

   ```py
   class loveyou:
   	love = 520

   # 就会输出'love': 520的键值对
   print(loveyou.__dict__)
   ```
7. `__doc__`：查看类的帮助文档

   ```py
   class loveyou:
   	"""
   	520you
   	"""
   # 就会输出520you，也就是帮助文档
   print(loveyou.__doc__)
   ```
8. `__class__`：返回对象所属的类
   可以用这个类来创建新的对象，如字符串对象`''`，`''.__class__`返回的就是str类，然后就可以用这个来创建新的字符串对象`''.__class__(1213)`，就创建了一个新的字符串对象`'1213'`
9. `__base__`：返回当前类的基类
   比如`str.__base__`会返回 **&lt;class 'object'&gt;**

##### 需要创建的方法

1. `__add__`、`__sub__`、`__mul__`、`__div__`、`__mod__`：算术运算，加减乘除模。
   对一个对象`a+b`会测试调用`a.__add__(b)`
   `__radd__`、`__rsub__`、`__rmul__`、`__rdiv__`、`__rmod__`：用来实现反向操作
   对一个对象b/a时会调用`a.__rdiv__(b)`
2. `__and__`，`__or__`、`__xor__`：逻辑运算，在实现逻辑运算时调用
3. `__len__`：返回对象的长度
   在使用`len(a)`时会调用`a.__len__()`

   ```py
   class A(object):
      def __len__(self):
          return 42

   a = A()
   print(len(a))
   ```
4. `__getitem__`：根据索引返回对象的元素
   在索引一个对象时会调用
   `a[1]`就是调用`a.__getitem__(1)`

​	`__setitem__`：相对的，会有这个方法用来索引赋值
​	在索引赋值一个对象时会调用
​	`a[1]=1`就是调用`a.__setitem__(1,1)`

5. `__import__`：载入模块的函数，
   `import os`  等价于 `os = __import__('os')`
6. `__builtins__`：包含了当前运行环境中默认的所有函数与类
   在pyjail中，这个一般被置为`None`

#### 重要的内置函数

1. `dir`：用来查看类的所有方法和属性，用来查看有什么可以利用的方法
2. `chr`、`ord`：字符与ascii码转换函数
3. `_`：这个是一个变量，返回的是上次运行的python语句结果，但是只在交互式终端才会产生，在运行代码文件时没用

### 远程代码执行（RCE）

RCE是指攻击者可以在一个组织的计算机或网络上运行恶意代码，执行攻击者控制的代码

#### 工作原理

* **注入漏洞**：如SQL注入或命令注入，因为应用不良的输入消毒产生的
* **不安全的反序列化**：序列化通过将数据集打包成单一的由接收方系统解包的位串，简化了数据集的传输。
  但是如果序列化数据的结构的定义不正确，攻击者可能可以制定一个解包时会被误解的输入，根据数据的存储和处理方式，这种误解可能会使攻击者能够实现代码执行
* **越界写入**：缓冲区是一块被分配用来存储数据的固定大小的内存，不安全的数据读取或写入可能允许攻击者将数据放在会被解释为代码或应用的重要控制信息的地方
* **文件管理**：有些应用允许用户向服务器上传文件，这种访问可能允许攻击者上传一个包含恶意代码的文件，并欺骗应用执行该代码

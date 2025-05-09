---
title: php
date: '2025-05-02 17:48:18'
updated: '2025-05-09 09:34:37'
permalink: /post/php-22mkms.html
comments: true
toc: true
---



# php

## php

### php基础语法知识

#### php标签写法

1. 正常标签
   ```php
   <?php 
   	echo "marin";
   ?>
   ```
2. 短标签
   在 `php.ini` 的设置中有一个选项 `short_open_tag = On` 的设置，当打开后就可以使用短标签
   这个写法更加灵活
   ```php
   <?  程序操作  ?>
   <?=函数?>
   <?="marin";?>   ===    <?php echo "marin";?>
   ```
3. asp写法
   同上，需要在 `php.ini` 中设置 `asp_tags = On`
   ```php
   <% echo 'marin'; %>
   ```
4. script脚本写法
   ```html
   ```

<scr<script language='php'>echo 'marin';</script>
```

#### 比较

* 弱比较
  php中的弱比较符号是 `==`  ，当php使用弱比较时，会进行类型转换，如果是数字和字符串相比较时，就会先将字符串转换成数值类型再进行比较
  当字符串不是数字开头时，会转换成0，否则则将字符串从转换成开头到第一个字母的数值
  ```
  admin1    ===>  0
  123admin  ===>  123
  ```

  - 科学计数法绕过
    php在遇到 `1e+数字` 这种类型时，会解析为科学计数法，然后计算出来
    这种方法可以用来绕过一些md5的比较，例如下面要传入两个不同但是md5值相同的值，这个很明显不可能，但是利用这个就能绕过
    只要传入两个md5值是 `0e` 开头的字符串就行，因为无论0的多少次方都是0

    ```php
    if($a != $b && md5($a) == md5($b))
    ```

    ```
    QNKCDZO
    s878926199a
    ```
* 强比较
  php中的强比较符号是 `===` ，主要是利用数组绕过

#### 数组绕过

在php中，绕过传入的是类似 `a[]=1&b[]=2`  这种的，php会自动解析成数组吗，即得到 `$a=[1]` ，这种可以绕过很多限制

- php的 `md5()` 和 `sha1()` 无法处理数组，可以绕过
- `strops()` 函数
  `strops` 函数查找字符串在另一字符串中第一次出现的位置（对大小写敏感）
  语法：`strops(string,find,start)`
- `strcmp()` 函数
  `strcmp()` 函数比较两个字符串（对大小写敏感）
  语法：`strcmp(string1,string2)`
- ……

### php文件包含

在php中，有文件包含的函数有

* include：直接将文件包含在当前代码当中
* include_once：只包含一次
* require：如果在包含过程中出错，就会直接退出，不执行后续语句
* require_once：只包含一次

### php伪协议

伪协议，又叫封装流协议，是php用来访问各种数据流的机制

* **file://** 
  访问本地文件系统

  ```
  http://www.xx.com?file=file:///home/marin/flag
  ```
* **php://filter**
  读取数据流

  ```
  http://www.xx.com?file=php://filter/read=convert.base64-encode/resource=
  ```

  > resource：必须的参数，指定要过滤筛选的数据流
  > read=：读链的过滤器
  > write=：写链的过滤器
  >
* **data://** 
  一般用来传入数据，例如写入 `file_get_content`
  与包含函数结合时，可以用来执行php代码

  ```
  data://text/plain,text  //这个//text/plain代表MIME类型，标识数据是文本
  ```
* **php://input**
  可以将POST请求的数据当作PHP代码执行

  ```
  http://www.xx.com?file=php://input

  [传POST] <?php phpinfo()?>
  ```
* **phar://** 
  这个是php解压缩的一个函数，不管后缀是什么，都可以当作压缩包解压，哪怕是txt文件，所以将木马文件压缩后，改成任意格式都可以正常使用

tips

* php在解析url伪协议时

  1. 检查类似php://filter关键字，确认是一个伪协议
  2. 然后就会查找`read=`或`write=`等参数，不会在意额外的路径片段

### php反序列化

为了方便将一个对象在网络上传输，可以将整个对象转化为二进制串，等到传输目的地时再将对象还原，这个过程就是序列化和反序列化

序列化：php中依靠的是`serialize()`
反序列化：php中依靠的是`unserialize()`

#### **序列化**</u>

属性类型
public：可以在任何地方被访问，在序列化后会变成`属性名`
protected：可以被自身及其子类和父类访问，在序列化后会变成`%00*%00属性名`
private：只能在定义的类中访问，在序列化后会变成`%00类名%00属性名`

```php
<?php
class fffff{
    public $name = "marin";
    protected $age = 18;
    private $sex = "male";
}

$a = new fffff();
echo serialize($a);
?>
```

在php中输出的序列化不会在其中加上`\00`或`%00`，需要手动加上，或者直接输出urlencode之后的数据

```php
O:5:"fffff":3:{s:4:"name";s:5:"marin";s:6:"*age";i:18;s:10:"fffffsex";s:4:"male";}
```

> `O`代表对象，`5`代表对象名长度为6，`People`为对象名，`3`代表有3个参数
>
> `{}`里是参数的key和value
>
> `s`代表strng对象，`4`代表长度，`name`代表key，后面同理
> 不同的数据类型有不同的字符

> 序列化格式
>
> a  ---   array  		 	 	数组型
> b  ---   boolean 	     	 	布尔型	
> d  ---   double 	       		 浮点型
> i   ---   integer 	       		 整数型
> o  ---   common object          	 共同对象
> r   ---   object reference 		 对象引用
> C  ---   custom object  		   自定义对象
> O  ---  class			 	   对象
> N  ---  null  				     空
> s   ---  non-escaped binary string   非转义的二进制字符串
> S  ---  escaped binary string           转义的字符串

序列化重点

1. 序列化只序列化属性，不序列化方法
2. 能控制得只有类的属性，攻击就是找合适得能被控制得属性，利用作用域本身存在的方法，发动攻击

####  **&lt;u&gt;**​**反序列化**​ **&lt;/u&gt;**

利用反序列化的前提，必须要有`unserialize()`函数，并且参数可控
在php中，一般反序列化都是创建一个新的对象（代码中已有的类的对象），所以可能会触发这个类中本身存在的魔术方法，导致触发反序列化漏洞

* php的魔术方法

|魔术方法|执行前提|调用|
| --------------| ------------------------------------------| ------------------------------------------------------------------------------------------------------------------------------|
|__wakeup()|当对象被反序列化时自动执行|使用unserizlize()反序列化一个对象时|
|__sleep()|当对象被序列化时自动执行|使用serialize()序列化一个对象时|
|__construct()|当对象被创建时自动执行|在new一个对象时被调用（可能会在构造pop链时被用上）|
|__destruct()|当对象被销毁时自动执行|一般是当脚本执行结束时才会销毁对象|
|__toString()|当对象被当作字符串使用时自动执行|1.被打印时触发（echo、print）<br />2.对象和字符串连接时<br />3.对象与字符串进行==比较时<br />4.对象经过php字符串函数时（strlen()、strtolower()）|
|__get()|访问不可访问属性时自动执行|访问一个不可访问的属性|
|__call()|访问不可访问方法时自动执行|调用一个不可访问的方法|
|__callStatic()|在静态上下文中调用不可访问的方法时自动执行|静态方法是使用`static`定义的，不需要实例化对象就能访问|
|__invoke()|将一个对象当作函数调用时自动执行|$object()  像这样时会触发|
|__set()|尝试给一个不可访问的属性赋值时自动执行|给一个不可访问的属性赋值|
|__isset()|对一个不可访问的属性上调用`issut()`或`empty()`|对一个不可访问的属性上调用`issut()`或`empty()`|
|__unset()|在不可访问的属性上使用`unset()`时触发|在不可访问的属性上使用`unset()`时触发|

#### pop链

pop链是从现有环境中找到一系列的代码或者指令调用，根据需求构成一组连续的调用链，最终达成攻击的目的

#### 绕过技巧

1. php7.1+的反序列化对类属性的检测不敏感
   所以对类中的`protected`修改成`public`也会被识别

   * 如果%00或\00被过滤可以尝试使用这个特性
2. __wakeup()绕过
   序列化的字符串中的**属性值个数**大于**属性个数**就会导致反序列化异常，从而绕过

   ```php
   O:5:"fffff":10:{s:4:"name";s:5:"marin";s:6:" * age";i:18;s:10:" fffff sex";s:4:"male";}
   ```
3. 16进制绕过字符过滤

   假设字符被过滤了，我们可以通过修改字符变为十六进制来绕过

   假设字符a被过滤了

   ```php
   O:5:"fffff":1:{s:4:"name";s:5:"marin";}
   =>
   O:5:"fffff":1:{S:4:"n\61me";S:5:"m\61rin";}
   将类型从s改为S，就会被当作十六进制解析
   ```

   ```php
   <?php

   class fffff{
       public $name = "marin";
   }

   $a = 'O:5:"fffff":1:{S:4:"n\61me";S:5:"m\61rin";}';

   $b = unserialize($a);
   echo $b->name;
   // marin
   ?>
   ```

#### 字符串逃逸

* 过滤后字符变多

漏洞成因：反序列化都是以`";}`结尾的，只要读取到了这个就会舍弃后面的内容
并且在反序列化时，php会根据s指定的字符串长度去读取后面的字符

原理：

```php
<?php
class marin{
public $name = "limbo";
public $key = "love";
}

$a = new marin();
$b = serialize($a);

$c = str_replace("limbo","hacker",$b);

echo $c;
?>
```

```php
O:5:"marin":2:{s:4:"name";s:5:"limbo";s:3:"key";s:4:"love";}

O:5:"marin":2:{s:4:"name";s:5:"hacker";s:3:"key";s:4:"love";}
```

这里`limbo`被替换成`hacker`之后，就逃逸了一个字符，这时候反序列化，name的值会是`hacke`

那如果我们将`name`的值在后面加上反序列化后的字符串，就能控制它的读取，我们在`name`的后面加上这一段内容，长度是27，我们就需要逃逸27个字符

```
";s:3:"key";s:7:"love123";}
```

我们就要在前面加上27个`limbo`

```
limbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbo";s:3:"key";s:7:"love123";}
```

```php
<?php
class marin{
public $name = 'limbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbo";s:3:"key";s:7:"love123";}';
public $key = "love";

}

$a = new marin();
$b = serialize($a);

$c = str_replace("limbo","hacker",$b);

echo $c;
$d = unserialize($c);
echo $d->key;

?>
```

然后就能输出

```php
O:5:"marin":2:{s:4:"name";s:162:"hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:3:"key";s:7:"love123";}";s:3:"key";s:4:"love";}
```

可以看到长度是name的s读取的长度是162，就是hacker的长度，刚好读取到我们加入的内容中，然后继续读取我们的内容进行反序列化

* 过滤后字符变少

相对于上面的变多

```php
<?php

class fffff{
    public $name = "marin";
    public $age = 18;
}

$a = 'O:5:"fffff":2:{s:4:"name";s:7:"marinxx";s:3:"age";i:18;}';
$a = str_replace('xx', 'x', $a);
echo $a;
$b = unserialize($a);
echo $b->age;
?>
```

将序列化字段中的字符替换少了，每替换一次，序列化后就会多读一个字符，这里就是利用的点

在原本的序列化字符串中，age读取的是个数字，并且替换后其实是不能输出的，会报错，但是如果我们在这里的age输入的是一个类似后面的字符串，并且根据数量将前面的给吞下，就会读取到我们输入的这个序列化字符串

```php
class fffff{
    public $name = "marinxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";
    public $age = '";s:3:"age";s:4:"male";}';
}
```

```php
O:5:"fffff":2:{s:4:"name";s:41:"marinxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx";s:3:"age";s:24:"";s:3:"age";s:4:"male";}";}
```

替换后变为，我们可以发现s读取的还是41个字符，而`marinxxxxxxxxxxxxxxxxxx`只有23个字符，所以他就会往后读取 `";s:3:"age";s:24:"`，将这一串都视为name属性

```php
O:5:"fffff":2:{s:4:"name";s:41:"marinxxxxxxxxxxxxxxxxxx";s:3:"age";s:24:"";s:3:"age";s:4:"male";}";}
```

在后面，我们输入的 `";s:3:"age";s:4:"male";}`，成功将前面的字符串闭合，并且能够读取到后面我们需要的值，将age的值从数字改为字符串male

> 区别
>
> 字符串变多只需要一个属性就可以完成
> 而字符串变少还需要另一个属性的配合

#### phar反序列化

* 要将php.ini中的`phar.readonly`改为off

  ```
  [Phar]
  https://php.net/phar.readonly
  phar.readonly = Off
  ```

这个是利用phar文件类型来进行反序列化的

* phar文件
  这个是php中类似JAR的一种打包文件，本质是一种压缩文件

  > 文件结构
  >
  > 1. stub
  >    标识这个是一个phar文件，__HALT_COMPILER();必须以这个结尾，前面内容不限
  > 2. manifest
  >    存储的是每个被压缩文件的权限、属性等信息，这部分还会以序列化形式存储用户自定义的meta-data，这里就是反序列化的漏洞点
  > 3. contents
  >    文件内容
  > 4. signature
  >    签名，验证文件
  >    签名结构
  >
  > |长度|内容|
  > | :-----| -------------------------------------------------------|
  > |看类型|sha1为20字节<br />md5为16字节<br />sha256为32字节<br />sha512为64字节|
  > |4字节|签名类型<br />1代表md5，2代表sha1，3代表sha256，4代表sha512，|
  > |4字节|GBMB标识|
  >

```php
<?php
    class getflag {
        function __destruct() {
            echo "FLAG";
        }
    }
    
    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
	
	$a = new getflag();
    $phar->setMetadata($a); //将自定义的meta-data存入manifest

    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

要讲到反序列化，还要学一个`phar://`伪协议

> `phar://`伪协议
> 这个是php解压缩的一个函数，不管后缀是什么，都可以当作压缩包解压，哪怕是txt文件，所以将木马文件压缩后，改成任意格式都可以正常使用

php一大部分文件系统的函数在通过`phar://`伪协议解析phar文件时，会将meta-data进行反序列化，从而进行攻击

|受影响的函数列表||||
| -----------------| -------------| ------------| -----------------|
|fileatime|filectime|file_exists|file_get_contents|
|file_put_contents|file|filegroup|fopen|
|fileinode|filemtime|fileowner|fileperms|
|is_dir|is_executable|is_file|is_link|
|is_readable|is_writable|is_writeable|parse_ini_file|
|copy|unlink|stat|readfile|
|mime_content_type||||

这些函数在读取到 `phar://` 伪协议时就可以反序列化

‍

‍

‍

参考
[PHP反序列化漏洞详解（万字分析、由浅入深）_php反序列化漏洞原理-CSDN博客](https://blog.csdn.net/Hardworking666/article/details/122373938)

[[CTF]PHP反序列化总结_ctf php反序列化-CSDN博客](https://blog.csdn.net/solitudi/article/details/113588692)

[PHP反序列化从初级到高级利用篇 - fish_pompom - 博客园](https://www.cnblogs.com/fish-pompom/p/11126473.html)

[PHP Phar反序列化总结_ctf phpphar反序列化-CSDN博客](https://blog.csdn.net/q20010619/article/details/120833148)

[PHP伪协议 – 4staR_x](https://www.4star.fun/2024/11/13/php伪协议/)

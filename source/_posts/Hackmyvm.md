---
title: Hackmyvm
date: '2025-05-05 09:33:16'
updated: '2025-05-08 23:55:25'
permalink: /post/hackmyvm-1dtzoz.html
comments: true
toc: true
---



# Hackmyvm

## Hackmyvm

---

~~好像跟哈希相关的一些题目~~ 得，还以为专门就是出哈希相关得题，后面其实差不多

### 005

---

拿到一个txt文件，题目提示要爆破

```
root:$6$xyz$ZGQOqL77wiYAgPxsNEv2Kz3INjzK4JdG29RbaHaW5lrkH8bA8W7kC3GK4CctGrFO7.E2va7kSgF3eQXNWYQee.:15758:0:99999:7:::
daemon:*:14971:0:99999:7:::
bin:*:14971:0:99999:7:::
sys:*:14971:0:99999:7:::
sync:*:14971:0:99999:7:::
games:*:14971:0:99999:7:::
man:*:14971:0:99999:7:::
lp:*:14971:0:99999:7:::
mail:*:14971:0:99999:7:::
news:*:14971:0:99999:7:::
uucp:*:14971:0:99999:7:::
proxy:*:14971:0:99999:7:::
www-data:*:14971:0:99999:7:::
backup:*:14971:0:99999:7:::
list:*:14971:0:99999:7:::
irc:*:14971:0:99999:7:::
gnats:*:14971:0:99999:7:::
nobody:*:14971:0:99999:7:::
libuuid:!:14971:0:99999:7:::
Debian-exim:!:14971:0:99999:7:::
```

主要的应该是root的内容，ai说前面的$6$表示是用sha-512加密的

但是对哈希这类的东西都不熟，研究一下

```bash
john timu/shadow.txt --wordlist=wordlist/rockyou.txt
```

![](https://pic.imgdb.cn/item/66ee7a61f21886ccc01abf52.png)

john是一个爆破密码的软件，这里的`[SHA512 256/256 AVX2 4x])`提示爆破的是一个512的密码哈希

[John - Powered by MinDoc (gui.run)](https://doc.gui.run/docs/security_tools/security_tools-1e06j26453ols)

|参数|意义|
| -----------| -----------------------------------------------------------------------|
|--single|使用简单破解模式|
|--wordlist=|用字典模式来进行加密|
|-rules|在字典模式下，开启字词规则变化功能，如读入hack，就会尝试hacker，hacke等|

这题也可以用hashcat破解，发现后面的也可以用hashcat，先研究一下这个

将txt中的哈希值保存下来，然后破解

这个查到是 `1800 | sha512crypt $6$, SHA512 (Unix)`

![](https://pic.imgdb.cn/item/66ee8771f21886ccc027c130.png)

### 008

---

拿到的是一个zip文件，但是什么信息都没给，想爆破，但是不知道密码类型

查的wp用了一个zip2john得到hash值

```
zip2john 008.zip

008.zip/flag.txt:$zip2$*0*3*0*751e06905814ebe63a63c72e8755d887*d807*e*25e3c7613e997071cd21a2163883*ba4cf18e59493b2515da*$/zip2$:flag.txt:008.zip:008.zip
```

试了下用john破解这个哈希好像不行

还是得用hashcat将哈希信息保存下来（一开始保存错了，前后都要删

```
$zip2$*0*3*0*751e06905814ebe63a63c72e8755d887*d807*e*25e3c7613e997071cd21a2163883*ba4cf18e59493b2515da*$/zip2$
```

![](https://pic.imgdb.cn/item/66ee8b1ef21886ccc02b7b91.png)

得到密码后就出flag了

### 011

---

蚌，一开始还没看出来东西，一缩小页面就看出来了

![](https://pic.imgdb.cn/item/66ee8babf21886ccc02c2281.png)

应该就是九键

对应一下得到flag

### 016

---

描述就是

```
HMV{\033[31mw\033[31mh\033[31my\033[31m i\033[31mm \033[31mn\033[31mo\033[31mt \033[39m m\033[39ma\033[39my\033[39mb\033[39me \033[39mu\033[39ms\033[39me \033[32m g\033[32mr\033[32me\033[32me\033[32mn\033[32mt\033[32mh\033[32me\033[32mf\033[32ml\033[32ma\033[32mg\033[39m}
```

没看出来是啥，给ai看了眼，说是ansi转义序列得字符串，但是ai给我转得是错的，查看发现 linux 中的 echo 语法可以输出这个字符串

![](https://pic.imgdb.cn/item/66ee8dcef21886ccc02e7bb2.png)

flag只有绿色那段

---
title: java学习
date: '2025-05-05 09:37:51'
updated: '2025-05-08 23:48:57'
permalink: /post/java-learning-lnqcv.html
comments: true
toc: true
---



# java学习

## JAVA

内存相关的先不学

---

### java概述

1. java是面向对象的语言
2. java是解释型的语言，编译后的代码不能直接被机器执行，需要解释器来执行
3. java是跨平台性的

#### java运行

##### 编译

```cmd
javac Hello.java
```

##### 运行

运行的其实是Hello的这个类，不需要.class

```cmd
java Hello
```

.java文件(源文件)   --(javac 编译)-->  .class文件(字节码文件)    --(java 运行)-->   结果

##### 反编译

```cmd
javap Hello.class
```

#### 注意事项

1. java程序的入口的是`main()`方法，有固定的书写的格式

```java
public static void main(String[] args)
```

2. java语言严格区分大小写
3. 一个源文件中，最多只能有一个public类，其他类的个数不限，每一个类编译都对应一个.class文件

> class Dog{}   会产生一个Dog.class文件

4. 如果源文件包含一个public的类，文件名必须是这个类名
5. 也可以将main方法卸载非public类中，然后指定运行非public类，这样入口方法就是非public的main方法

#### java的内存

1. 栈：方法运行时使用的内存，方法进栈运行，运行完就出栈
2. 堆：存放对象（数组），new创建的都存储在堆内存
3. 方法区：存储可以运行的class文件

![](https://pic1.imgdb.cn/item/678a09c4d0e0a243d4f52c04.png)

值传递和引用传递的区别

值传递是将原先的值的内存拷贝一份，放到新变量的地址中
引用传递是将新的变量指向原先的变量的地址

#### java API

java语言提供了大量的基础类，有些API文档用来记录如何使用
[DoubleAdder - Java17中文文档 - API参考文档 - 全栈行动派](https://doc.qzxdp.cn/jdk/17/zh/api/java.base/java/util/concurrent/atomic/DoubleAdder.html)

#### idea

##### idea项目结构

* 项目 project 

  * 模块 model

    * 包 package

      * 类 class

##### idea快捷键

​`ctrl + H`​          	查看一个类的层级关系  
​`ctrl + B`​          	定位方法的位置  
​`alt  + enter`​   	自动导入类  
​`aaaa.var`​		句后加`.var`​自动分配变量名  
​`alt + insert`​   	自动写一个函数  
​`ctrl + alt + L`​  	自动格式化代码  
​`alt + 鼠标滚轮`​	竖着选中

##### idea模板

1. ​`main`​
2. ​`sout`​    输出模板
3. ​`fori`​      循环模板

### 基础语法

#### 转义字符

1. ​`\t`​：制表位，实现对齐的功能
2. ​`\n`​：换行符
3. ​`\\`​：一个`\`​
4. ​`\"`​：一个`"`​
5. ​`\'`​：一个`'`​
6. ​`\r`​：一个回车，回车表示回到最开头，再输出内容，就会替换原本的内容

```java
System.out.println("北京天津\r上海");
// 最后会输出上海天津
```

#### 注释

1. 单行注释   //
2. 多行注释   /*      */
3. 文档注释   注释的类容可以被jdk根据`javadoc`解析，生成一套以网页形式体现的说明文档
   这里添加的是javadoc标签

   ```java
   /**
    * @author  mairn
    * @version 1.0
   */
   ```

   ```
   javadoc -d 文件夹 -xx -yy Comment.java
   ```

   ![](https://pic1.imgdb.cn/item/67887604d0e0a243d4f4b07f.png)

#### java代码规范

1. 类和方法的注释要用javadoc的方法来写
2. 源文件使用utf-8编码

#### 变量

java的变量和c一样是在内存中分配一个地址，将值存储到这个地址中，变量就指向这个地址

> +号的运用  
> 1.当左右两边都是数值型的时候，做加法运算  
> 2.当左右两边有一个是String字符串的时候，做拼接运算  
> 3.char类型的加法是Unicode码相加  
> 4.按顺序从左到右

##### 数据类型

###### 基本数据类型

数据存储在自己的空间中  
特点：赋值给其他变量，也是赋的真实的值

* 数值型

  * 整数类型

  > ​`byte`​   	[1]
  >
  > ​`short `​ 	[2]
  >
  > ​`int`​        	[4]
  >
  > ​`long`​      	[8]
  >

  使用细节

  1. java的整形默认为 `int` 型，声明long型常量要加`l`或`L`

     ```java
     long a = 9999L;
     ```
  2. 变量一般就用 int ，除非不足以表示大数，才使用long

  * 浮点类型
    浮点数 = 符号位+指数位+尾数位
    尾数部分可能丢失，造成精度损失

  > ​`float`​    	  	[4]
  >
  > ​`double`​ 		[8]
  >

  使用细节

  1. 默认为 `double` 型，声明 `float` 型要在后面加`f`或`F`，但是在double的后面加也没问题

     ```java
     float a = 123.123F;
     ```
  2. 表示形式
     十进制表示法：`5.12`     `512.0f`      `.512`  (必须要有小数位)
     科学计数法：5.12e2   [5.12*10的2次方]    5.12E-2  [5.12除以10的2次方]
  3. `2.7`和`8.1/3`的比较不是相等
     `8.1/3`会是一个接近2.7的小数，而不是2.7
     在java中，对运算结果是小数的值进行相等判断是会有问题的
     所以应该用两个值的差值的绝对值，在一个精度范围内进行判断

     ```java
     double num1 = 2.7;
     double num2 = 8.1/3;

     if(Math.abs(num1 - num2) < 0.00000001){
     	System.out.println("数值相等");
     }
     ```
* 字符型

  > `char`    [2]
  >

  使用细节

  1. 字符常量使用`''`括起来的单个字符（`""`双引号括起来是字符串类型）
  2. 用`\`来将字符变为转义字符
  3. java中，char的本质是一个整数，输出会变成 对应的Unicode字符

     ```java
     char c1 = 97;
     System.out.println(c1);   // a 

     char c2 = 'a';
     System.out.println((int)c2);   // 97
     ```
  4. char类型是可以计算的，相当于一个整数
* 布尔型

  > `boolean`   [1]
  >

  使用细节

  1. 和c语言不一样，不可以用0或非0整数替代false和true

###### 引用数据类型

变量中存储的是地址值，数据是存储在其他空间中的  
特点：赋值给其他变量，赋的是地址值

* 类（`class`）

  > String算类的数据类型
  >
* 接口（`interface`）
* 数组（`[]`）

##### 自动类型转换

在java中，进行赋值或者运算时，精度小的会自动转换成精度大的数据类型

![](https://pic1.imgdb.cn/item/6788e4a0d0e0a243d4f4df46.png)

![](https://pic1.imgdb.cn/item/6788ec57d0e0a243d4f4e1c3.png)

```java
int a = 'c';    // c是char类型，可以转换成int

double d = 80;  // 80是int类型，可以转换成double
```

1. 有多种类型的数据混合计算的时候，系统先将所有数据转换成数据中精度最高的那种数据类型
2. 精度大的赋值给精度小的就会报错，反之会自动转换
3. ​`byte`​ `short`​ `char`​ 三者是可以计算的，它们在计算时会**先转换为** **​`int`​**​ **类型**
4. 表达结构的类型会自动提示为操作数中最大的类型

##### 强制类型转换

加上一个强制转换符`()`，但是会造成精度降低或溢出

1. 数据从大到小的转换时，就会用到强制转换
2. 强制符号只针对最近的操作数有效

   ```java
   int x = (int)10*1.1+10*12;
   // 报错，int只转换了10，后面的类型是double，大于int的精度

   int y = (int)(10*1.1+10*12);
   // 正确
   ```

##### 基本数据类型和String类型的转换

* 基本数据类型转换为String类型  
  基本数据类型的值加`""`​就行  

  ```java
  int n1 = 100;
  float  f1 = 1.1F;
  double d1 = 1.1;

  String s1 = n1 + "";
  String s2 = f1 + "";
  String s3 = d1 + "";
  ```

* String类型转换为基本数据类型
  通过基本类型的包装类调用parseXX方法

  ```java
  String s1 = "123";

  int n1 = Integer.parseInt(s1);
  double d1 = Double.parseDouble(s1);
  float f1 = Float.parseFloat(s1);
  long l1 = Long.parseLong(s1);
  byte b1 = Byte.parseByte(s1);
  short s1 = Short.parseShort(s1);

  boolean bool = Boolean.parseBoolean("Ttue");
  ```
* String转换为char

  ```java
  String s1 = "1223";

  System.out.println(s1.charAt(0));
  ```

##### 作用域

1. 在java中，主要的变量就是全局变量和局部变量
2. 局部变量就是在方法中定义的变量
3. java中作用域的分类  
    **全局变量**：属性，作用域为整个类体，可以被本类和其他类使用。属性可以添加修饰符  
    **局部变量**：除了属性以外的变量，作用域为定义它的代码块中（一个大括号就是一个代码块），只能在本类对应的方法中使用。不可以添加修饰符
4. 全局变量可以不赋值直接使用，因为有默认值  
    局部变量必须赋值后使用，没有默认值
5. 全局变量和局部变量可以重名，使用的时候遵循就近原则

    ```java
    public class marin{
    	public int name;
    	public void medthod(){
    		int name = 10;
    		System.out.printIn(name);  // 10
    	}
    }
    ```

#### 运算符

1. 算术运算符

|运算符|效果|
| ------| ------------------------------------------------|
|+|正号|
|-|负号|
|+|加法|
|-|减法|
|*|乘法|
|/|除法|
|%|取余|
|++|自增（前）：先自增后赋值<br />自增（后）：先赋值后自增|
|--|自减（前）：先自减后赋值<br />自减（后）：先赋值后自减|

> 一些特殊的例子
>
> 1. 除法
>
> ```java
> System.out.println(10 / 4); // 2
> double a1 = 10 / 4;   // 2
> double a2 = 10.0 /4;  // 2.5
> ```
>
> 2. 加法  
>     ​`char`​ 类型的 + 数字类型  
>     会先将字符转换成 ascii 对应的数字再计算
> 3. 取余  
>     本质是一个公式 ： a % b = a - a / b * b
>
> ```java
> System.out.println(10 % 3);    // 1
> System.out.println(-10 % 3);   // -1
> System.out.println(10 % -3);   // 1
> System.out.println(-10 % -3);  // -1
> ```
>
> 3. 自增
>
> ```java
> int j = 8;
> int k = ++j; 
> // 等价于   j = j + 1;  k = j;
> // k = j = 9   
> int x = j++;
> // 等价于   x = j; j = j + 1;
>  // x = 9 j = 10
> ```
>
> 特殊
>
> ```java
> int i = 1;
> i = i++; 
> /*
> 	会使用一个临时变量
> 	1.temp = i;
> 	2.i = i + 1;
> 	3.i = temp;
> 	i = 1
> */
> ```
>
> ```java
> int i = 1;
> i = ++i; 
> /*
> 	会使用一个临时变量
> 	1.i = i + 1;
> 	2.temp = i;
> 	3.i = temp;
> 	i = 2
> */
> ```

2. 关系运算符
   结果都是boolean类型

|运算符|运算|
| ----------| --------------------------------|
|==|相等|
|!=|不等|
|<|小于|
|>|大于|
|<=|小于等于|
|>=|大于等于|
|instanceOf|判断对象是否为XX类或XX类的子类型|

> **instanceOf**
> 语法：对象 instanceof 类名
> 特点：判断的是**运行类型**

3. 逻辑运算符
   连接多个条件，结果也是boolean

|a|b|a&b|a&&b|a\|b|a\|\|b|!a|a^b|
| ---| ---| -----| ------| ---------| --------------| ----| -----|
|T|T|T|T|T|T|F|F|
|T|F|F|F|T|T|F|T|
|F|T|F|F|T|T|T|T|
|F|F|F|F|F|F|T|F|

> & 逻辑与              && 短路与
>
> 区别：
>
> && 短路与如果第一个条件为F，则不会判断第二个条件，效率高  
> &    逻辑与两个条件都要判断，效率低
>
> ‍
>
> | 逻辑或                || 短路或
>
> 区别：
>
> || 短路或如果第一个条件为T，则不会判断第二个条件，效率高  
> |  逻辑或两个条件都要判断，效率低

4. 赋值运算符

    ​`+=`​   `-=`​   `*=`​   `%=`​

    复合赋值运算符会就行类型转换

    ```java
    byte b = 3;
    b += 2;
    // 等价 b = (byte)(b + 2)
    b++;
    // 等价 b = (byte)(b + 1)
    ```
5. 三元运算符

    ​`条件表达式 ? 表达式1 : 表达式2;`​

    1. 如果条件表达式为T，运算后的结果是表达式1
    2. 如果条件表达式为F，运算后的结果是表达式2
    3. 表达式1和表达式2需要是能赋值的类型或可以自动转换

    ```java
    int a = 10;
    int b = 99;
    int result = a > b ? a++:b--;
    // a = 10,b = 98, result = 99
    ```

6. 运算符优先级
7. 位运算

   原码，反码，补码

   1. 二进制最高位为符号位，0正1负
   2. 正数的三种码都一样
   3. 负数反码为    **符号位不变，其他位取反**
   4. 负数补码为    **负数反码加一**
   5. java没有无符号数
   6. 计算机运算的时候都是用**补码**来运算的，结果要看**原码**

   |位运算符|规则|
   | -----------------| ----------------------------------|
   |按位与 &|与规则|
   |按位或 \||或规则|
   |按位异或  ^|异或规则|
   |按位取反  ~|取反规则|
   |算术右移 >>|低位溢出，符号位不变，用符号位补|
   |算术左移 <<|符号位不变，低位补0|
   |无符号右移  >>>|低位溢出，高位补0|

‍

#### 标识符

对各种变量，类名和方法的命名使用的字符串称为标识符

1.不能用数字开头
2.不可以使用关键字和保留字
3.严格区分大小写
4.不能包含空格

#### 输出语句

```java
System.out.println();  // 自动换行   
System.out.print();    // 不会自动换行
```

#### 输入语句

java中获取数据，需要一个扫描器

```java
// 1.导入 Scanner 类所在的包
import java.util.Scanner; 

// 2.创建 Scanner 对象
Scanner myScanner = new Scanner(System.in);

// 3.接受输入
String name = myScanner.next(); 
int age = myScanner.nextInt(); 
double score = myScanner.nextDouble(); 
```

#### 关键字

##### this

java虚拟机会给每个对象分配一个this，用来代表**当前对象**(应该是相当于python的self)

1. this可以访问本类的属性(成员变量)，方法，构造器
2. this可以区分当前类的属性和局部变量
3. 访问构造器语法：`this(参数列表);`​  
   只能在构造器中使用，并且只能是第一条语句

   ```java
   class T{
       String name;
   	public T(){
           // 访问另一个构造器，
           this("marin");
   	}
   	public T(String name){
           this.name = name;
       }
   }
   ```
4. 只能在类定义的方法中使用

##### super

代表父类的引用，引用访问父类的属性，方法，构造器

1. 访问属性

   ```java
   super.属性名;
   ```
2. 访问父类的方法

   ```java
   super.方法名(参数列表);
   ```
3. 访问父类的构造器(只能第一句)

   ```java
   super(参数列表);
   ```

super的访问会一直向上访问，遵循就近原则

#### 控制结构

##### 顺序控制

从上到下地执行，没有任何判断和跳转

##### 分支控制

**if-else**

```java
if(条件表达式1){
	执行代码块1;
}
else if(条件表达式2){
	执行代码块2;
}
……
else{
    执行代码块n;
}
```

**switch**

细节
1.数据类型应该和case的常量类型一致，或者能自动转换
2.表达式的返回值必须是（byte、short、int、char、enum[枚举]、String）
3.没有break会穿透到下一个case

```java
switch(表达式){
    case 常量1:
        语句块1;
        break;
    case 常量2:
        语句块2;
        break;
	……
    case 常量n:
        语句块n;
        break;
    default:
        语句块;
        break;
}
```

**比较**

1. 如果判断的具体数值不多，而且符合（byte、short、int、char、enum[枚举]、String），使用switch会好一点
2. 对区间判断，对结果为boolean类型的判断，使用if

##### 循环控制

**for**

细节
1.循环条件是一个返回boolean值的表达式
2.循环变量初始化和循环变量迭代可以省略，但是两边的分号不能省略
3.循环条件省略就进入一个死循环
4.循环变量初始化和循环变量迭代可以有多条语句，中间用逗号隔开

```java
for(循环变量初始化;循环条件;循环变量迭代){
    循环操作;
}
```

‍

**while**

细节
1.while先判断再执行语句

```java
循环变量初始化;
while(循环变量){
	循环体;
	循环变量迭代;
}
```

‍

**do..while**

细节
1.do.while先执行后判断，至少执行一次
2.最后会有一个分号

```java
do{
	循环体;
	循环变量迭代;
}while(循环条件);
```

‍

##### 跳转控制语句

**break**

终止某个语句块的执行，一般使用在switch或者循环中

细节
1.break语句出现在多层嵌套语句块中，可以用**标签**指明要终止的是哪一层语句块

```java
label1：{
label2:    	{
label3:    		{
                 	break label2;   
                 }   
			}
		}
```

2.没有标签，默认退出最近的循环体

**continue**

结束本次循环，继续下一个循环

细节
1.continue和break一样，可以用标签指明跳出哪一层循环

**return**

表示跳出所在方法

‍

#### 数组

数组用来存放多个**同一类型**的数据，是**引用数据类型**，数组型数据是对象  
数组在存储数据时会有自动类型转换  
数组在创建后，如果没有赋值，会有默认值

> ​`int`​			0  
> ​`short`​	   	0		  
> ​`byte`​	     	0  
> ​`long`​ 	    	0  
> ​`float`​	     	0.0  
> ​`double`​         	0.0  
> ​`char`​	     	\u0000  
> ​`boolean`​       	false  
> ​`String`​	   	null
>
> ​`引用数据类型`​	null

**数组定义**

1. 动态初始化

​`数据类型[]   数组名 = new 数据类型[大小]`​

​`数据类型     数组名[] = new 数据类型[大小]`​

```java
int a[] = new int[5] 
```

2. 动态初始化

先声明数组        `数据类型 数组名[];`​

再创建数组        `数组名 = new 数据类型[大小];`​

```java
int a[];                // 这时候a在内存中是空的，还没分配空间
a = new int[10]; 	    // new 给a分配内存空间
```

3. 静态初始化

​`数据类型 数组名[]  =  {元素值，元素值....}`​

```java
int[] array = {1,1,2,3,4};
// 可以通过 array[下标] 来访问数组的元素
```

‍

**数组遍历**

```java
for(int i = 0;i < 5;i++){
    System.out.println("第" + i + "个元素的值" + array[i]);
}
```

‍

**数组长度**

可以用`数组名.length`获取数组的长度

‍

**数组赋值**

数组默认情况下是引用传递，赋值的是地址，赋值方式为引用传达

```java
int[]  arr1 = {1,2,3};
int[]  arr2 = arr1;
// 对arr2的修改会影响arr1
```

‍

##### 多维数组

**二维数组**

```java
int[][] arr = {{0,0,0},{1,1,1},{0,0,1}};
```

和一维数组一样可以用`数组名.length`获取数组的长度

**数组定义**

声明方式

```java
int[][] y
int y[][]  
int[] y[]
```

1. 动态初始化

​`数据类型[] []   数组名 = new 数据类型[大小] [大小]`​  
（第二个大小可以省略）

```java
int a[][] = new int[5][5]; 
```

2. 动态初始化

先声明数组        `数据类型 数组名[] []`​;

再创建数组        `数组名 = new 数据类型[大小] [大小]`​;

```java
int a[][];                // 这时候a在内存中是空的，还没分配空间
a = new int[10][2]; 	    // new 给a分配内存空间
```

3. 静态初始化

​`数据类型 数组名[]  = {{},{},{}}`​

```java
int[] array = {{1},{1,2},{3,4}};
// 可以通过 array[下标] 来访问数组的元素
```

‍

#### 方法

方法是程序中最小的执行单元

**创建**

```java
class Object1{
    String a;
    public void method(int n){
		System.out.print(n);
    }
    public int getSum(int num1,int num2){
        int res = num1 + num2;
        return res;
    }
}
```

**使用**

当程序执行方法时，会在栈中开辟一个独立的空间  
当方法执行完毕，或执行到return语句，就会返回到调用方法的地方  
返回后，继续执行调用方法后的代码  
当main方法（main栈）执行完毕，整个程序退出

```java
// 不同的类中
Object1 O1 = new Object1();
O1.method();  // 调用方法
```

```java
// 同一个类中
class A{
    public void printA(String a){
        System.out.println(a);
	}
    public void test(){
		String A = "123"
         printA(A);
    }
}
```

**细节**

1. 一个方法最多只有一个返回值，要返回多个数据，要用数组
2. 如果方法要求有返回数据类型，最后的执行语句必须为return，且与返回数据类型一致或兼容
3. 返回类型为void的没有或者只写 `return ;`​
4. 调用带参数的方法时，一定要传入一致或兼容的参数
5. 方法定义的参数叫 形式参数（形参），方法调用时传入的参数称为 实际参数（实参）
6. 同一个类中的方法直接调用，在不同的类中，就要通过对象名调用

##### **方法重写**

子类的方法和父类的一个方法，名称、返回类型、参数一样，就说这个方法重写了父类的方法

1. 子类方法的返回类型可以是和父类方法返回类型一致，或者是父类返回类型的子类  
    比如 父类是 object 则子类可以是 String
2. 子类方法不能缩小父类方法的访问权限，但是可以扩大

##### **递归调用**

递归就是方法自己调用自己

1. 在栈中每次递归都会创建一个新的空间（栈空间）
2. 方法的局部变量是独立的，不会相互影响
3. 方法中如果使用的是引用类型变量，就会共享该引用类型的数据

##### **方法重载**

java中允许在同一个类中，多个同名方法的存在，但是要求**形参列表不一致**（类型，顺序，个数）（参数名和返回类型不算）

```java
// 例如,这个方法有很多不同的形参,但是方法名字一致
public static void PrintA(int i){
	System.out.println(i);
}
public static void PrintA(char i){
	System.out.println(i);
}
public static void PrintA(short i){
	System.out.println(i);
}
```

##### 可变参数

java中，可以将同一个类中的**多个同名同功能**但**参数个数不同**的方法，封装成一个方法，通过可变参数实现

语法：`访问修饰符 返回类型 方法名(数据类型... 形参名){ }`​

```java
class Math{
    public int sum(int... nums){
        // int... ：表示接受的是可变参数，类型是int，可以接受多个int
        // 使用参数时，可以当作数组使用，即 nums 当作数组使用
    }    
}
```

可变参数可以和普通类型的参数一起放在形参列表，但是可变参数在最后，且一个形参列表最多只能出现一个可变参数

### 面向对象

**类与对象（OOP）**

**类** 就是自定义的数据类型，**对象** 就是用类创建的

#### 类

**定义类**​

```java
public class 类名{
	1. 成员变量
	2. 成员方法
	3. 构造器
	4. 代码块
	5. 内部类
}
```

补充：

* 用来描述一类事物的类，叫做：**javabean类**  
  在 javabean类中，是不写 main 方法的
* 编写 main 方法的类，叫做：**测试类**  
  可以在测试类中创建 javabean类 对象并进行赋值调用
* 一个java文件可以定义多个 class类，且只能一个类是 public 修饰，而且 public 修饰的类名必须成为**代码文件名**

##### **成员变量**

使用 `对象.成员变量`​ 来赋值或使用

1. 成员变量是类的一个组成部分
2. 成员变量的定义同语法变量，例子：  
    ​`访问修饰符 属性类型 成员变量名`​

    ```java
    protected String name;
    ```
3. 成员变量如果不赋值，有默认值，规则和数组一致

##### **成员方法**

使用 `对象.方法`​ 调用

#### 对象

**创建对象**

1.先声明再创建

```java
Cat cat;
cat = new Cat();
```

2.直接创建

```java
Cat cat = new Cat();
```

```java
public class Object{
	public static void main(String[] args){
		// 实例化一只猫
		Cat cat1 = new Cat();
		cat1.name = "小白";
		cat1.age = 3;

		Cat cat2 = new Cat();
		cat2.name = "小花";
		cat2.age = 100;

		System.out.println(cat1.name+" "+cat1.age);
		System.out.println(cat2.name+" "+cat2.age);
	}
}
class Cat {
	// 属性
	String name;
	int age;
}
```

对象也是一个引用类型  
对象会存在堆中，对象名指向堆中的一个空间，空间中对象的属性如果是字符串，会指向方法区中的一个常量池  
实例化一个类的时候，会在方法区加载类的信息  
（1）属性信息  
（2）方法信息

![](https://pic1.imgdb.cn/item/678a53a6d0e0a243d4f54445.png)

#### 访问修饰符

控制方法和属性的访问权限

1. ​`public`​	        公开级别，对外公开
2. ​`protected`​      受保护级别对子类和同一个包中的类公开
3. ​`默认`​                没有修饰符号，对同一个包的类公开
4. `private`​         	私有级别，只有类本身可以访问，不对外公开

|访问修饰符|同类|同包|子类|不同包|
| ----------| ----| ----| ----| ------|
|`public`|√|√|√|√|
|`protected`|√|√|√|×|
|`默认`|√|√|×|×|
|`private`|√|×|×|×|

**细节**：

1. 修饰符可以修饰类中的属性，方法以及类
2. 只有`默认`和`public`才能修饰类

‍

#### 特征

面向对象有三大特征：封装、继承、多态

##### 封装

把抽象出的数据 `属性`​ 和对数据的操作 `方法`​ 封装在一起  
数据被保护在内部，程序的其他方法只有通过被授权的操作 `方法`​ ，才能对数据进行操作

**步骤**

1. 将属性私有化 `private`​
2. 提供一个公共的 set 方法，用于对属性判断并赋值
3. 提供一个公共的 get 方法，用于获取属性的值

    * 可以使用 快捷键  **alt + insert**  选择Getter和Setter来快速写这两个方法  
      ​![](https://pic1.imgdb.cn/item/678db4d4d0e0a243d4f5c65b.png)
4. 如果有构造器，就将set写到构造器中

##### 继承

解决代码复用，当多个类存在相同的属性和方法时，抽象出父类，在父类中定义这些相同的属性和方法

**语法**：

```java
class 子类 exends 父类{
}
```

**细节**：

1. 子类必须先调用父类的构造器，先完成父类的初始化，再调用自己的构造器
2. 创建子类对象时，不过使用哪个构造器，都会去调用父类的无参构造器，如果父类没有提供无参构造器，则必须再子类的构造器中使用  **super**  去指定使用父类的哪个构造器完成对父类的初始化工作，否则编译无法通过

    ```java
    public class Sub extends Base{
        public Sub(){
    		super(); // 这个是默认存在的，不用自己写
        }
        public Sub(String name){
            // 如果父类没有无参构造器，就必须要有这个super
            super(参数);
        }
    }
    ```
3. 如果希望调用父类的某个构造器，需要用  **super**  去指定
4. **super**  使用时必须在代码体中的第一行
5. **super**  和  **this**  都必须在第一行，所以一个构造器只能使用其中一个
6. 父类构造器的调用不限于直接父类，可以一直向上追溯直到object类
7. 子类最多只能继承一个父类，java中是**单继承制**，要多继承只能一直往上叠
8. 当子类调用一个属性/方法时

    1. 当子类有这个属性/方法，并且可以访问，则返回这个属性
    2. 子类没有这个属性/方法，就找父类
    3. 如果父类没有，则按照2的规则查找上级父类，直到Object或找到

##### 多态

方法或对象具有多种形态，多态是建立在封装和继承基础之上的

1. 方法的多态  
    方法的重写和重载就体现方法的多态
2. 对象的多态（核心）

    1. 一个对象的编译类型和运行类型可以不一致

        ```java
        Animal animal = new Dog();
        // animal 的编译类型是 Animal ，运行类型是Dog

        animal = new Cat();
        // animal 的运行类型变成了Cat，编译类型不变
        ```
    2. 编译类型在定义对象时，就确定了
    3. 运行类型是可以变化的
    4. 编译类型看定义时 = 的左边，运行类型看 = 的右边  
        调用方法的时候查看编译类型的方法，运行这个方法的时候查看运行类型是如何实现的

        （运行类型就是这个对象指向的地址空间的类型）

```java
// 使用多态机制
// animal 编译类型是Animal，可以接收Animal子类的对象，就不用每个对象写一个方法
public void feed(Animal animal,Food food) {
	System.out.println("主人" + Name + "给" +
		animal.getName() + "吃" + food.getName());
}
```

**细节**：

1. 多态的前提是：两个对象（类）存在继承关系
2. 在**编译阶段**，能调用哪些成员，由编译类型决定的  
    在**运行阶段**，要看运行类型的实现
3. 属性没有重写的说法，**属性**只看**编译类型**  
    （感觉其实属性应该就是在编译阶段就决定了，不会被运行阶段影响）
4. 多态的向上转型  
    本质：父类的引用指向了子类的对象  
    语法：父类类型  引用名  =  new  子类类型();  
    特点：可以调用父类中的所有成员（但是要遵守访问权限）  
    不可以调用子类中的特有成员（子类特有的方法或变量）  
    最终运行的结构，要看**子类（运行类型）** 的具体实现（如果父类的方法被子类重新写了，就会使用子类的方法）
5. 多态的向下转型  
    语法：子类类型 引用名 = （子类类型） 父类引用  
    特点：只能强制转换父类的引用不能强制转换父类的对象  
    要求父类的引用必须指向的是当前目标类型的对象（就是这个父类引用的运行类型得是当前转换的类型）  
    向下转型后可以调用子类类型中的所有成员

    ```java
    // 向上转型
    // 编译类型是Animal，运行类型是Cat
    Animal animal = new Cat();

    // 向下转型
    // 编译类型是Cat，运行类型是Cat
    // 要求是animal之前的引用必须是指向Cat
    Cat cat = (Cat) animal;
    ```

**java的动态绑定机制**

1. 当调用对象方法的时候，该方法会和该对象的**内存地址/运行类型**绑定

    ```java
    // 编译类型是A,运行类型是B
    A a = new B();
    // 运行类型没有sum(),所以先调用A的sum()方法
    // A的sum()中出现了getI(),这个方法在a的运行类型B中存在,所以调用B的getI()
    System.out.println(a.sum()); // 30


    class A {
    	public i = 10;

        public int sum(){
            return getI() + 10;
        }
    	public int getI(){
    		return i;
        }
    }

    class B extends A{
        public i = 20;

        public int getI(){
            return i;
        }
    }

    ```
2. 当调用**对象属性**时，**没有动态绑定机制**，哪里声明，那里使用

**多态数组**

数组的定义类型是父类类型，里面保存的实际元素为子类类型

如果要调用子类类型的特有方法，要使用强制转换，将数组中的元素转换为对应的子类

‍

‍

#### 构造器（构造方法）

创建一个对象时，由虚拟机自动调用，完成对对象的初始化

语法：`修饰符 类名（形参列表）{ 方法体; }`​

```java
// 调用
Person P1 = new Person("marin",18); 

// 构造器的创建
class Person{
	String name;
	int age;
	
	// 构造器
	public Person(String name,int age){
		this.name = name;
		this.age = age;
	}
}
```

1. 构造器没有返回值，也没有返回值类型，void 都没有
2. 方法名和类名必须一致
3. 在创建对象时，系统会自动调用该类的构造器完成对象的初始化，每创建一个对象，就会调用一次
4. 一个类可以定义多个不同的构造器
5. 如果没有定义构造器，系统会自动生成一个默认的**无参构造器**，可以使用 javap 来反编译出来

    如果定义了一个有参构造器，这个默认的无参构成器就会消失，这个时候如果不带参就会报错，除非多定义一个**无参构造器**

    ```java
    Person(){};
    ```

    ![](https://pic1.imgdb.cn/item/678ca8e2d0e0a243d4f59e62.png)

#### 包

包的本质就是创建不同的文件夹来保存不同的类

作用：区分相同名字的类
管理类
控制访问范围

语法：

```java
// package 关键字，表示打包，用来声明当前类所在的包，需要放在类的最上面
package 包名
```

命名：包的命名一般是
不能以数字开头，不能是关键字或保留字

```
com.项目名.业务模块名
```

在idea中右键src，新建软件包，即可创建包

![](https://pic1.imgdb.cn/item/678ceef4d0e0a243d4f5ab77.png)

![](https://pic1.imgdb.cn/item/678cef1bd0e0a243d4f5ab7c.png)

**常用的包**

​`java.lang`​    	默认引用，  
​`java.util`​      系统提供的工具包，工具类  
​`java.net`​       	网络包，网络开发  
​`java.awt`​      	做java的页面开发，GUI

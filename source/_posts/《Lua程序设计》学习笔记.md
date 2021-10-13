---
title: 《Lua程序设计》学习笔记
date: 2021-10-13
description: 
cover: https://cdn.jsdelivr.net/gh/sakuya-kai/cloudimg/img/skadi.jpg
---

### 命令行执行

类型为`table`的全局变量`arg`保存命令行参数，脚本名为`arg[0]`，真正的参数从`arg[1]`开始，而脚本之前则是负数的下标

例如`lua -e "sin=math.sin" script a b`的参数如下
```
arg[-3] = "lua"
arg[-2] = "-e"
arg[-1] = "sin=math.sin"
arg[0] = "script"
arg[1] = "a"
arg[2] = "b"
```

---

### 八种基本数据类型

- `nil`
- `Boolean`
- `number`
- `string`
- `userdata`
- `function`
- `thread`
- `table`

`Lua`是一个动态类型语言，不用声明变量的类型，每个变量携带自己的类型

`type(v)` 获取`v`的变量类型，返回的是字符串
```
type(a)             --> nil
a = 1 type(a)       --> number
a = "1" type(a)     --> string
a = nil type(a)     --> nil
type(type(a))       --> string
```

#### 逻辑运算符

```
and
or
not
```

骚操作
> 表达式`x and y`，当`x`为`false`时值为`x`，为`true`时值为`y`  
> 表达式`x or y`，当`x`为`false`时值为`y`，为`true`时值为`x`  
> 表达式`x and y or z`，当`x`为`false`时值为`z`，为`true`时值为`y`(`y`不能为`false`)

**注意**：在条件判断上，只有`Boolean`的`false`和`nil`是假，其他任何值都是真，包括数字`0`和空字符串`""`

---


### Numbers

从`Lua 5.3`开始，`number`类型有两种存储方式，分别是`64-bit integer`和双精度浮点型`float`，之前只有双精度浮点型，在有限制的平台上可以把`Lua 5.3`编译成`Small Lua`，使用`32-bit integer`和单精度浮点型`float`

`type(num)`只能输出`number`，如果要区分实际的类型，可以用`math.type()`

```
 > math.type(3) 	--> integer
 > math.type(3.0) 	--> float
```

#### 算术运算符

```
+
-
*
/		  除法，两个操作数会先转成`float`再运算，保证跟`float`的除法一致
//  	  整除，两个`integer`结果还是`integer`
%
^		  幂运算
```


```
 > 3.0 / 2.0	--> 1.5
 > 3 / 2 		--> 1.5
 
 > 3 // 2 		--> 1
 > 3.0 // 2 	--> 1.0
 > 6 // 2 		--> 3
 > 6.0 // 2.0 	--> 3.0
 > -9 // 2 		--> -5
 > 1.5 // 0.5 	--> 3.0
```

取模骚操作，获取浮点数精确到几位的数

```
 > x = math.pi
 > x - x % 0.01 	--> 3.14
 > x - x % 0.001 	--> 3.141
```



#### 关系运算符

```
>
<
<=
>=
==
~=
```



#### `math`库

```
math.pi				数字π
math.huge				最大的number，输出inf
math.random()			生成[0,1)的数
math.random(n)			生成[1,n]的数
math.random(a,b)			生成[a,b]的数
math.randomseed(os.time())		方便的设置随机种子
math.maxinteger			最大integer
math.mininteger			最小integer
math.tointeger(num)		浮点型转整型，如果不能转换就返回nil
```

`num + 0.0` 整型转浮点型

`num | 0` 浮点型转整型，不能有小数部分，且数值大小必须在范围内，否则会报错

浮点型能精确表示的整型范围`[-2^51, 2^51]`




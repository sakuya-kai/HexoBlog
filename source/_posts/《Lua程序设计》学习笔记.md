---
title: 《Lua程序设计》学习笔记
date: 2021-10-13
description: 菜鸟的Lua学习之路
cover: https://cdn.jsdelivr.net/gh/sakuya-kai/cloudimg/img/skadi.jpg
tags: Lua
---

### 命令行执行

类型为`table`的全局变量`arg`保存命令行参数，脚本名为`arg[0]`，真正的参数从`arg[1]`开始，而脚本之前则是负数的下标

例如`lua -e "sin=math.sin" script a b`的参数如下
```lua
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
```lua
type(a)             --> nil
a = 1 type(a)       --> number
a = "1" type(a)     --> string
a = nil type(a)     --> nil
type(type(a))       --> string
```

#### 逻辑运算符

```lua
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

```lua
 math.type(3) 	--> integer
 math.type(3.0) 	--> float
```

#### 算术运算符

```lua
+
-
*
/		  --除法，两个操作数会先转成float再运算，保证跟float的除法一致
//  	  --整除，会舍弃小数部分，就算两个操作数里面有float类型的数
%
^		  --幂运算
```


```lua
 3.0 / 2.0	--> 1.5
 3 / 2 		--> 1.5
 
 3 // 2 		--> 1
 3.0 // 2 	--> 1.0
 6 // 2 		--> 3
 6.0 // 2.0 	--> 3.0
 -9 // 2 		--> -5
 1.5 // 0.5 	--> 3.0
```

取模骚操作，获取浮点数精确到几位的数

```lua
 x = math.pi
 x - x % 0.01 	--> 3.14
 x - x % 0.001 	--> 3.141
```



#### 关系运算符

```lua
>
<
<=
>=
==
~=
```



#### `math`库

```lua
math.pi				--数字π
math.huge				--最大的number，输出inf
math.random()			--生成[0,1)的数
math.random(n)			--生成[1,n]的数
math.random(a,b)			--生成[a,b]的数
math.randomseed(os.time())		--方便的设置随机种子
math.maxinteger			--最大integer
math.mininteger			--最小integer
math.tointeger(num)		--浮点型转整型，如果不能转换就返回nil
```

`num + 0.0` 整型转浮点型

`num | 0` 浮点型转整型，不能有小数部分，且数值大小必须在范围内，否则会报错

浮点型能精确表示的整型范围`[-2^51, 2^51]`

---

### Strings

`Lua`的字符串也是**不可变类型**

字符串的定义用**双引号**和**单引号**都行

长字符串定义，里面的字符串不会被转义，两个`[[`和`]]`之间可以加**任意数量**的`=`，避免`a[b[i]]`这种混乱，可以为0个

```lua
s = [==[
abcde
\n\n
haha
]==]
```

`#s` 取字符串字节长度，注意是字节长度，在不同编码下可能跟字符长度不一样

```lua
a = "hello"
print(#a)			--> 5
print(#"good bye")		--> 8
print(#"哈哈")		--> 4
```

`s1 .. s2`拼接字符串，`s1`是`number`类型时，后面必须要跟一个空格，不然编译器会把第一个点当成数字的小数点，从而报错

```lua
"Hello " .. "World" 	--> Hello World
"result is " .. 3 		--> result is 3
1 .. 1			--> 11
```

#### 字符串与数字的转换

任何算术操作，如果操作数是个字符串，会尝试把字符串转成数字

相反，任何字符串操作，如果操作数是数字，会尝试把数字转成字符串

**注意**：字符串转数字的结果都会转成浮点型`float`

```lua
"10" + 1 			--> 11.0
```

显示地把字符串转换成数字

```lua
tonumber(s)		--默认按10进制转换
tonumber(" -3 ") 		--> -3
tonumber(" 10e4 ") 	--> 100000.0
tonumber("10e") 		--> nil (not a valid number)
tonumber("0x1.3p-4") 	--> 0.07421875

tonumber(s, base)		--按照指定进制base转换，返回都是10进制，2 <= base <= 32
tonumber("100101", 2) 	--> 37
tonumber("fff", 16) 	--> 4095
tonumber("-ZZ", 36) 	--> -1295
tonumber("987", 8) 	--> nil
```

显示地把数字转换成

```lua
tostring(num)
```

#### `string`库

```lua
string.len(s)		--获取字符串长度，跟#str一样
string.rep(s, n) 		--重复字符串str n次
string.reverse(s) 		--反转字符串
string.lower(s) 		--转大写
string.upper(s) 		--转小写
string.sub(s, i, j)	--获取子字符串，[i, j]，闭区间，下标从1开始，可以是负数，-1是最后一个字符，-2是倒数第二个字符，以此类推

s = "[in brackets]"
string.sub(s, 2, -2) 	--> in brackets
string.sub(s, 1, 1) 	--> [
string.sub(s, -1, -1) 	--> ]

string.char(a, b, c,...)	--把数字转换成对应的ASCII字符
print(string.char(97)) 	--> a

string.byte(s, i)		--把字符串s的第i个字符转成对应的ASCII数字，i参数可选，不填就是转第一个字符
print(string.byte("abc")) 	--> 97
print(string.byte("abc", 2)) --> 98
print(string.byte("abc", -1)) --> 99

string.byte(s, i, j)	--转换[i, j]闭区间的字符串
print(string.byte("abc", 1, 2)) --> 97 98
string.byte(s, 1, -1)	--转换所有

string.format(s, v1, v2,...)	--格式化字符串

string.find(s, p)		--模式匹配，返回匹配到的第一个位置和最后一个位置，匹配不到返回nil
string.find("hello world", "wor") --> 7 9
string.find("hello world", "war") --> nil

string.gsub(s, old, new)	--用new替换s中所有的old，第一个返回值是替换后的字符串，第二个返回值是替换的数量
string.gsub("hello world", "l", ".") --> he..o wor.d 3
string.gsub("hello world", "ll", "..") --> he..o world 1
string.gsub("hello world", "a", ".") --> hello world 0
```

#### Tips

所有字符串库方法，可以用`s:fun()`这种格式来执行

```lua
s = "123"
print(s:len())		--> 3
```

`reverse,upper,lower,byte,char`这几个方法对`utf-8`编码的字符串可能会失效

`len,sub`取的是字节长度而不是字符长度



### Tables

`Lua`里主要的（实际上是唯一的）数据结构机制

本质上是一个联合数组，不过索引不止是数字，还可以是其他任何值，除了`nil`（emmm，这不就是hash表吗）

```lua
a = {}		--创建表
a[1]		--> nil	访问未初始化的值
a[1] = 2	--赋值
a[1]		--> 2	

a.x			--> nil	a.x == a["x"] 语法糖 注意 a.x ~= a[x]
a.x = 10
a.x			--> 10

a[2.0] = 1
a[2]		--> 1 只要key相等，integer和float能访问同一个值，实际上，任何能转换成integer的float，在key上都会先转换成integer再进行操作
```

#### 多种构造方式

```lua
a = {}	--1.初始化一个空表，最基础的方式

days = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"}	--2.初始化直接赋值，按下标从1开始
days[4] --> Wednesday

a = {x = 10, y = 20}	--3.key-value初始化
a = {}; a.x = 10; a.y = 20	--跟上面结果一样，不过上面性能好一些，因为在创建表的时候就知道正确的大小

b = {x = 10, y = 20, 30, 40, z = 50, 60}	--4.混合初始化， b[1] = 20, b[2] = 30, b[3] = 60

c = {["+"] = 10, ["-"] = 20, [-1] = 30}	--5.增强行的key-value初始化，可以突破按值初始化不能用负下标和普通的key-value初始化不用能特殊key的限制，复杂但是更灵活

opnames = {["+"] = "add", ["-"] = "sub", ["*"] = "mul", ["/"] = "div"}
i = 20; s = "-"
a = {[i+0] = s, [i+1] = s..s, [i+2] = s..s..s}
print(opnames[s]) --> sub
print(a[22]) --> ---

{x = 0, y = 0} -- {["x"] = 0, ["y"] = 0}	
{"r", "g", "b"} -- {[1] = "r", [2] = "g", [3] = "b"}

a = {1; 2}	--间隔也可以用分号，不过现在已经很少用了
```

####  Arrays, Lists, and Sequences

用表作为一个数组

```lua
-- read 10 lines, storing them in a table
a = {}
for i = 1, 10 do	-- 按照惯例，Lua的数组从下标1开始
    a[i] = io.read()
end
```

中间没有空洞，这样的数组可以称为`Sequence`

```lua
#a	--> 10 获取数组的长度，不能对有空洞的数组获取长度，实际上是[1, n]的key所能到达的最大n的值
a[#a + 1] = v	-- 在数组最后添加一个值
```

#### 遍历表

用`pairs`迭代器遍历，但是遍历顺序不确定，每次执行可能都是不同的顺序

```lua
t = {10, print, x = 12, k = "hi"}
for k, v in pairs(t) do
    print(k, v)
end
--> 1 10
--> k hi
--> 2 function: 0x420610
--> x 12
```

对于数组，可以用`ipairs`迭代器，会保证顺序

```lua
t = {10, print, 12, "hi"}
for k, v in ipairs(t) do
    print(k, v)
end
--> 1 10
--> 2 function: 0x420610
--> 3 12
--> 4 hi
```

遍历数组的另一种方式

```lua
t = {10, print, 12, "hi"}
for k = 1, #t do
    print(k, t[k])
end
--> 1 10
--> 2 function: 0x420610
--> 3 12
--> 4 hi
```

#### 安全访问

比起普通的`if lib and lib.foo then...`，用`a or {}`这种方式效率更高，能访问更少次数的表，虽然写法比较难看

```lua
zip = company and company.director and	-- 访问了6次表
company.director.address and
company.director.address.zipcode

zip = (((company or {}).director or {}).address or {}).zipcode	-- 只访问3次表，跟某些语言的company?.director?.address?.zipcode相似

E = {} -- can be reused in other similar expressions
...
zip = (((company or E).director or E).address or E).zipcode
```

#### `table`库

可以理解成`List`或`Sequence`的库..

```lua
table.insert(t, i, v)	-- 在i的位置插入v，把后面的值往后移一位
t = {10, 20}
table.insert(t, 1, 30)	--> {30, 10, 20}
table.insert(t, v)	-- 不指定位置，会直接插在最后面，其他值不会位移

table.remove(t, i)	-- 删除i位置的值并返回该值，后面的值往前移一位
table.remove(t)	-- 删除最后一个值

table.move(t, i, j, k)	-- 把t的元素中[i, j]下标的移动到k下标

table.move(t, 1, #t, 2)	-- 在表头添加一个元素
t[1] = newElement

table.move(t, 2, #t, 1)	-- 删除第一个元素
t[#t] = nil

table.move(t1, i, j, k, t2) -- 把t1中[i, j]下标的元素复制到t2的k下标

table.move(t1, 1, #a, 1,{})	-- 复制表
table.move(t1, 1, #t1, #t2 + 1, t2)	-- 把t1表添加到t2后面
```



### Functions












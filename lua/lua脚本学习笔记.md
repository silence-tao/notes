# lua脚本学习笔记

[学习链接](https://www.runoob.com/lua/lua-data-types.html)

## 1.环境安装

### 1.1 Mac OS X 系统上安装

**下载安装包方式安装**

```
curl -R -O http://www.lua.org/ftp/lua-5.3.0.tar.gz
tar zxf lua-5.3.0.tar.gz
cd lua-5.3.0
make macosx test
make install
```

**直接试用 BrewHome 安装**

``` shell
brew install lua
```

安装信息：

``` bash
chentao10@SHMAC-C02GK0ZYM ~ % brew install lua
Running `brew update --preinstall`...
==> Auto-updated Homebrew!
Updated 1 tap (homebrew/core).
==> New Formulae
libapplewm      mbw             mkfontscale     primecount      stanc3
libxfont2       minimap2        monika          sse2neon        xkbcomp
==> Updated Formulae
Updated 148 formulae.

==> Downloading https://ghcr.io/v2/homebrew/core/lua/manifests/5.4.4_1
######################################################################## 100.0%
==> Downloading https://ghcr.io/v2/homebrew/core/lua/blobs/sha256:55abe1007284d3
==> Downloading from https://pkg-containers.githubusercontent.com/ghcr1/blobs/sh
######################################################################## 100.0%
==> Pouring lua--5.4.4_1.big_sur.bottle.tar.gz
==> Caveats
You may also want luarocks:
  brew install luarocks
==> Summary
🍺  /usr/local/Cellar/lua/5.4.4_1: 29 files, 750.6KB
==> Running `brew cleanup lua`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```

## 2.基本语法

### 2.1 交互式编程

Lua 提供了交互式编程模式。我们可以在命令行中输入程序并立即查看效果。

Lua 交互式编程模式可以通过命令 lua -i 或 lua 来启用：

```bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua -i
Lua 5.4.4  Copyright (C) 1994-2022 Lua.org, PUC-Rio
>
```

在命令行中，输入以下命令:

```lua
> print("Hello World")
```

接着我们按下回车键，输出结果如下：

``` lua
> print("Hello World")
Hello World
> 1 + 1
2
```

### 2.2 脚本式编程

我们可以将 Lua 程序代码保存到一个以 lua 结尾的文件，并执行，该模式称为脚本式编程，如我们将如下代码存储在名为 hello.lua 的脚本文件中：

```lua
print("Hello World！")
print("www.silentao.com")
```

使用 lua 名执行以上脚本，输出结果为：

```bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua hello.lua 
Hello World！
www.runoob.com
```

我们也可以将代码修改为如下形式来执行脚本（在开头添加：`#!/usr/local/bin/lua`）：

``` lua
#!/usr/local/bin/lua

print("Hello World")
print("www.silentao.com")
```

以上代码中，我们指定了 Lua 的解释器 `/usr/local/bin` directory。加上 # 号标记解释器会忽略它。接下来我们为脚本添加可执行权限，并执行：

```bash
chentao10@SHMAC-C02GK0ZYM lua-practical % chmod +x hello01.lua
chentao10@SHMAC-C02GK0ZYM lua-practical % ./hello01.lua  
Hello World
www.silentao.com
```

### 2.3 注释

**单行注释**

两个减号是单行注释:

```lua
--
```

**多行注释**

```lua
--[[
 多行注释
 多行注释
 --]]
```

------

### 2.4 标示符

Lua 标示符用于定义一个变量，函数获取其他用户定义的项。标示符以一个字母 A 到 Z 或 a 到 z 或下划线 **_** 开头后加上 0 个或多个字母，下划线，数字（0 到 9）。

最好不要使用下划线加大写字母的标示符，因为Lua的保留字也是这样的。

Lua 不允许使用特殊字符如 **@**, **$**, 和 **%** 来定义标示符。 Lua 是一个区分大小写的编程语言。因此在 Lua 中 Silence 与 silence 是两个不同的标示符。以下列出了一些正确的标示符：

```lua
mohd         zara      abc     move_name    a_123
myname50     _temp     j       a23b9        retVal
```

### 2.5 关键词

以下列出了 Lua 的保留关键词。保留关键字不能作为常量或变量或其他用户自定义标示符：

| and      | break | do    | else   |
| -------- | ----- | ----- | ------ |
| elseif   | end   | false | for    |
| function | if    | in    | local  |
| nil      | not   | or    | repeat |
| return   | then  | true  | until  |
| while    | goto  |       |        |

> 一般约定，以下划线开头连接一串大写字母的名字（比如 _VERSION）被保留用于 Lua 内部全局变量。

### 2.6 全局变量

在默认情况下，变量总是认为是全局的。

全局变量不需要声明，给一个变量赋值后即创建了这个全局变量，访问一个没有初始化的全局变量也不会出错，只不过得到的结果是：nil。

``` lua
> print(b)
nil
> print(a)
nil
> a = 2
> print(a)
2
```

如果你想删除一个全局变量，只需要将变量赋值为 nil。

``` lua
> a = nil
> print(a)
nil
```

这样变量 b 就好像从没被使用过一样。换句话说, 当且仅当一个变量不等于 nil 时，这个变量即存在。

## 3. Lua数据类型

Lua 是动态类型语言，变量不要类型定义,只需要为变量赋值。 值可以存储在变量中，作为参数传递或结果返回。

Lua 中有 8 个基本类型分别为：nil、boolean、number、string、userdata、function、thread 和 table。

| 数据类型 | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| nil      | 这个最简单，只有值 nil 属于该类，表示一个无效值（在条件表达式中相当于 false）。 |
| boolean  | 包含两个值：false 和 true。                                  |
| number   | 表示双精度类型的实浮点数                                     |
| string   | 字符串由一对双引号或单引号来表示                             |
| function | 由 C 或 Lua 编写的函数                                       |
| userdata | 表示任意存储在变量中的 C 数据结构                            |
| thread   | 表示执行的独立线路，用于执行协同程序                         |
| table    | Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字、字符串或表类型。在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。 |

我们可以使用 type 函数测试给定变量或者值的类型：

``` lua
print(type("Hello world"))      --> string
print(type(10.4*3))             --> number
print(type(print))              --> function
print(type(type))               --> function
print(type(true))               --> boolean
print(type(nil))                --> nil
print(type(type(X)))            --> string
```

### 3.1 nil（空）

nil 类型表示一种没有任何有效值，它只有一个值 -- nil，例如打印一个没有赋值的变量，便会输出一个 nil 值：

``` lua
> print(a)
nil
```

对于全局变量和 table，nil 还有一个"删除"作用，给全局变量或者 table 表里的变量赋一个 nil 值，等同于把它们删掉，执行下面代码就知：

``` lua
tab1 = { key1 = "val1", key2 = "val2", "val3" }
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end
 
tab1.key1 = nil
for k, v in pairs(tab1) do
    print(k .. " - " .. v)
end
```

执行结果：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_nil.lua 
1 - val3
key1 - val1
key2 - val2
1 - val3
key2 - val2
```

nil 作比较时应该加上双引号 「"」：

``` lua
> type(a)
nil
> type(a) == nil
false
> type(a) == "nil"
true
```

`type(a) == nil` 结果为 **false** 的原因是 type(a) 实质是返回的 **"nil"** 字符串，是一个 string 类型：

```lua
> type(type(a))
string
```

### 3.2 boolean（布尔）

boolean 类型只有两个可选值：true（真） 和 false（假），Lua 把 false 和 nil 看作是 false，其他的都为 true，数字 0 也是 true:

``` lua
print(type(true))
print(type(false))
print(type(nil))

if false or nil then
    print("false和nil至少一个是true")
else
    print("false和nil都是false")
end

if 0 then
    print("数字0是true")
else
    print("数字0是false")
end
```

以上代码执行结果如下：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_boolean.lua 
boolean
boolean
nil
false和nil都是false
数字0是true
```

### 3.3 number（数字）

Lua 默认只有一种 number 类型 -- double（双精度）类型（默认类型可以修改 luaconf.h 里的定义），以下几种写法都被看作是 number 类型：

``` lua
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```

以上代码执行结果：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_number.lua 
number
number
number
number
number
number
```

### 3.4 string（字符串）

```lua
-- 字符串可以由一对双引号或单引号来表示
string1 = "this is string1"
string2 = 'this is string2'
print(string1)
print(string2)

-- 也可以用 2 个方括号 "[[]]" 来表示"一块"字符串。
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```

以上代码执行结果为：

``` bash
this is string1
this is string2
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
```

在对一个数字字符串上进行算术操作时，Lua 会尝试将这个数字字符串转成一个数字:

``` lua
> print("2" + 6)
8
> print("2" + "6")
8
> print("2 + 6")
2 + 6
> print("-2e2" * "6")
-1200.0
> print("error" + 1)
stdin:1: attempt to add a 'string' with a 'number'
stack traceback:
	[C]: in metamethod 'add'
	stdin:1: in main chunk
	[C]: in ?
```

以上代码中 "error" + 1 执行报错了，字符串连接使用的是 .. ，如：

``` lua
> print("a" .. "b")
ab
> print(100 .. 200)
100200
```

使用 # 来计算字符串的长度，放在字符串前面，如下实例：

``` lua
> len = "www.silentao.com"
> print(#len)
16
> print(#"www.silentao.com")
16
```

### 3.5 table（表）

在 Lua 里，table 的创建是通过"构造表达式"来完成，最简单构造表达式是{}，用来创建一个空表。也可以在表里添加一些数据，直接初始化表:

``` lua
-- 创建一个空的 table
local tbl1 = {}
 
-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}
```

Lua 中的表（table）其实是一个"关联数组"（associative arrays），数组的索引可以是数字或者是字符串。

``` lua
-- lua_table.lua 脚本文件
a = {}
a["key"] = "value"

key = 10
a[key] = 20
a[key] = a[key] + 12

for k, v in pairs(a) do
    print(k .. " : " .. v)
end
```

脚本执行结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_table.lua 
10 : 32
key : value
```

不同于其他语言的数组把 0 作为数组的初始索引，在 Lua 里表的默认初始索引一般以 1 开始。

``` lua
local tbl = {"apple", "pear", "orange", "grape"}

for key, val in pairs(tbl) do
    print(key .. " : " .. val)
end
```

脚本执行结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_table02.lua 
1 : apple
2 : pear
3 : orange
4 : grape
```

table 不会固定长度大小，有新数据添加时 table 长度会自动增长，没初始的 table 都是 nil。

``` lua
a = {}
for i = 1, 10 do
    a[i] = i
end

a["key"] = "val"

print(a["key"])
print(a["none"])
```

脚本执行结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_table03.lua 
val
nil
```

### 3.6 function（函数）

在 Lua 中，函数是被看作是"第一类值（First-Class Value）"，函数可以存在变量里:

``` lua
function func(n)
    if n == 0 then
        return 1
    else 
        return n * func(n - 1)
    end
end

print(func(5))
funcCopy = func
print(funcCopy(5))
```

脚本执行结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_function.lua
120
120
```

function 可以以匿名函数（anonymous function）的方式通过参数传递:

``` lua
function testFun(tab, fun)
    for k, v in pairs(tab) do
        print(fun(k, v))
    end
end

tab = {key1 = "value1", key2 = "value2"}

testFun(tab,
    function(key, val) -- 匿名函数
        return key .. " = " .. val
    end
)
```

脚本执行结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_function01.lua
key2 = value2
key1 = value1
```

### 3.6 thread（线程）

在 Lua 里，最主要的线程是协同程序（coroutine）。它跟线程（thread）差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分东西。

线程跟协程的区别：线程可以同时多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起（suspend）时才会暂停。

### 3.8 userdata（自定义类型）

userdata 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和 指针）存储到 Lua 变量中调用。

## 4.Lua 变量

变量在使用前，需要在代码中进行声明，即创建该变量。

编译程序执行代码之前编译器需要知道如何给语句变量开辟存储区，用于存储变量的值。

Lua 变量有三种类型：**全局变量**、**局部变量**、**表中的域**。

**Lua 中的变量全是全局变量，哪怕是语句块或是函数里，除非用 local 显式声明为局部变量**。

局部变量的作用域为从声明位置开始到所在语句块结束。

变量的默认值均为 nil。

``` lua
a = 5           -- 全局变量
local b = 5   -- 局部变量

function joke()
    c = 5       -- 全局变量
    local d = 6 -- 局部变量
end

joke()

print(c, d)     --> 5 nil

do
    local a = 6 -- 局部变量
    b = 6       -- 全局变量 
    print(a, b) --> 6 6
end

print(a, b)     --> 5 6
```

执行以上实例输出结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_variable.lua
5	nil
6	6
5	6
```

### 4.1 赋值语句

赋值是改变一个变量的值和改变表域的最基本的方法。

``` lua
a = "hello" .. "world"
t.n = t.n + 1
```

Lua 可以对多个变量同时赋值，变量列表和值列表的各个元素用逗号分开，赋值语句右边的值会依次赋给左边的变量。

``` lua
a, b = 10, 2*x       -->       a=10; b=2*x
```

遇到赋值语句Lua会先计算右边所有的值然后再执行赋值操作，所以我们可以这样进行交换变量的值：

``` lua
x, y = y, x                     -- swap 'x' for 'y'
a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'
```

当变量个数和值的个数不一致时，Lua会一直以变量个数为基础采取以下策略：

> a. 变量个数 > 值的个数             按变量个数补足nil
> b. 变量个数 < 值的个数             多余的值会被忽略

``` lua
a, b, c = 0, 1
print(a, b, c)          --> 0 1 nil

a, b = a + 1, b + 1, b + 2      -- b + 2 会被忽略
print(a, b)             --> 1 2

a, b, c = 0
print(a, b, c)          --> 0 nil nil
```

执行以上实例输出结果为：

``` bash
chentao10@SHMAC-C02GK0ZYM lua-practical % lua lua_variable02.lua 
0	1	nil
1	2
0	nil	nil
```

上面最后一个例子是一个常见的错误情况，注意：如果要对多个变量赋值必须依次对每个变量赋值。

``` lua
a, b, c = 1, 2, 3
print(a, b, c)          --> 1 2 3
```

多值赋值经常用来交换变量，或将函数调用返回给变量：

``` lua
a, b = f()
```

f() 返回两个值，第一个赋给 a，第二个赋给 b。

应该尽可能的使用局部变量，有两个好处：

- 避免命名冲突。
- 访问局部变量的速度比全局变量更快。

### 4.2 索引

对 table 的索引使用方括号 []。Lua 也提供了 . 操作。

``` lua
t[i]
t.i                 -- 当索引为字符串类型时的一种简化写法
gettable_event(t,i) -- 采用索引访问本质上是一个类似这样的函数调用
```

实际操作：

``` lua
> site = {}
> site["key"] = "www.silentao.com"
> print(site["key"])
www.silentao.com
> print(site.key)
www.silentao.com
```




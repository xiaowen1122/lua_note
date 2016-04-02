lua_basic.md
========
LuaJIT 2.0.4 -- Copyright (C) 2005-2015 Mike Pall. http://luajit.org/

from:https://moonbingbing.gitbooks.io/openresty-best-practices

## 基本数据类型
函数`type`返回一个值或一个变量所属的类型

```
print(type("hello world")) -->output:string
print(type(print))         -->output:function
print(type(true))          -->output:boolean
print(type(360.0))         -->output:number
print(type(nil))           -->output:nil
print(type({}))            -->output:table
```

1. nil

 `nil`表示无效值。一个变量第一次赋值前的默认值为`nil`，将`nil`赋予给一个变量等同于删除它。
 OpenResty的Lua接口还提供了一种特殊的空值，`ngx.null`，用来表示不同于`nil`的“空值”。

2. boolean
 
 可选值为`true`或`false`。Lua中，`nil`和`false`为“假”，其它值都为“真”。比如0和空字符串是“真”。

3. number

 Number 类型用于表示实数，和 C/C++ 里面的 double 类型很类似。可以使用数学函数 math.floor（向下取整）和 math.ceil（向上取整）进行取整操作。
 一般地，Lua 的 number 类型就是用双精度浮点数来实现的。值得一提的是，LuaJIT 支持所谓的“dual-number”（双数）模式，即 LuaJIT 会根据上下文用整型来存储整数，而用双精度浮点数来存放浮点数。另外，LuaJIT 还支持“长长整型”的大整数。
 
4. string

 Lua中有三种方式表示字符串：
 1. 使用一对匹配的单引号，如`'hello'`
 2. 使用一对匹配的双引号，如`"hello"`
 3. 使用长括号(`[[]]`)括起来 ，如`[[hello]]`。

		两个正的方括号（即`[[`）间插入 n 个等号定义为第 n 级正长括号。 就是说，0 级正的长括号写作`[[`， 一级正的长括号写作`[=[`，如此等等。 反的长括号也作类似定义； 举个例子，4 级反的长括号写作`]====]`。 一个长字符串可以由任何一级的正的长括号开始，而由第一个碰到的同级反的长括号结束。 整个词法分析过程将不受分行限制，不处理任何转义符，并且忽略掉任何不同级别的长括号。 这种方式描述的字符串可以包含任何东西，当然本级别的反长括号除外。 例：`[[abc\nbc]]`，里面的 `\n` 不会被转义。

	```
	local str1 = 'hello world'
	local str2 = "hello lua"
	local str3 = [["add\name",'hello']]
	local str4 = [=[string have a [[]].]=]
	
	print(str1)    -->output:hello world
	print(str2)    -->output:hello lua
	print(str3)    -->output:"add\name",'hello'
	print(str4)    -->output:string have a [[]].
	```

5. table

	Table 类型实现了一种抽象的“关联数组”。“关联数组” 是一种具有特殊索引方式的数组，索引通常是字符串（string）或者 number 类型，但也可以是除 nil 以外的任意类型的值。
	
	```
	local corp = {
	    web = "www.google.com",   --索引为字符串，key = "web",
	                              --            value = "www.google.com"
	    telephone = "12345678",   --索引为字符串
	    staff = {"Jack", "Scott", "Gary"}, --索引为字符串，值也是一个表
	    100876,              --相当于 [1] = 100876，此时索引为数字
	                         --      key = 1, value = 100876
	    100191,              --相当于 [2] = 100191，此时索引为数字
	    [10] = 360,          --直接把数字索引给出
	    ["city"] = "Beijing" --索引为字符串
	}
	
	print(corp.web)               -->output:www.google.com
	print(corp["telephone"])      -->output:12345678
	print(corp[2])                -->output:100191
	print(corp["city"])           -->output:"Beijing"
	print(corp.staff[1])          -->output:Jack
	print(corp[10])               -->output:360
	```
	
	注意上例中的`corp[2]`索引。数字下标从1起，且是从第一个没有key的value起。
	
6. function

	在 Lua 中，函数 也是一种数据类型，函数可以存储在变量中，可以通过参数传递给其他函数，还可以作为其他函数的返回值。有名函数的定义本质上是匿名函数对变量的赋值。
	```
	local function foo()
	end
	```
	等价于
	```
	local foo = function ()
	end
	```
	
## 表达式
### 算术运算符
加`+`,减`-`,乘`*`,除`/`,指数`^`,取模`%`

```
print(5/10)      -->0.5
print(2^10)      -->1024
```

### 关系运算符
小于`<`，大于`<`，小于等于`<=`，大于等于`>=`，等于`==`，不等于`~=`
结果为`boolean`类型。`==`比较时是引用比较，也就是只有两个变量引用同一个对像时才相等。
```
local a = { x = 1, y = 0}
local b = { x = 1, y = 0}
if a == b then
  print("a==b")
else
  print("a~=b")
end

---output:
a~=b
```

### 逻辑运算符
逻辑与`and`，逻辑或`or`，逻辑非`not`
- `a and b` 如果 a 为 nil，则返回 a，否则返回 b;
- `a or b` 如果 a 为 nil，则返回 b，否则返回 a。
所有逻辑操作符将 false 和 nil 视作假，其他任何值视作真，对于 and 和 or，“短路求值”，对于not，永远只返回 true 或者 false。

### 字符串连接

在 Lua 中连接两个字符串，可以使用操作符“..”（两个点）。如果其任意一个操作数是数字的话，Lua 会将这个数字转换成字符串。注意，连接操作符只会创建一个新字符串，而不会改变原操作数。也可以使用 string 库函数 string.format 连接字符串。

```
print("Hello " .. "World")    -->打印 Hello World
print(0 .. 1)                 -->打印 01

str1 = string.format("%s-%s","hello","world")
print(str1)              -->打印 hello-world

str2 = string.format("%d-%s-%.2f",123,"world",1.21)
print(str2)              -->打印 123-world-1.21
```

由于 Lua 字符串本质上是只读的，因此字符串连接运算符几乎总会创建一个新的（更大的）字符串。这意味着如果有很多这样的连接操作（比如在循环中使用 .. 来拼接最终结果），则性能损耗会非常大。在这种情况下，推荐使用 table 和 table.concat() 来进行很多字符串的拼接，例如：

```
local pieces = {}
for i, elem in ipairs(my_list) do
    pieces[i] = my_process(elem)
end
local res = table.concat(pieces)
```

当然，上面的例子还可以使用 LuaJIT 独有的 table.new 来恰当地初始化 pieces 表的空间，以避免该表的动态生长。

### 优先级
1. ^
2. not,#,-
3. *,/,%
4. +,-
5. ..
6. <,>,<=,>=,==,=
7. and
8. or

## 控制结构
1. if/else

	```
	score = 0
	if score == 100 then
	    print("Very good!Your score is 100")
	elseif score >= 60 then
	    print("Congratulations, you have passed it,your score greater or equal to 60")
	else
	    if score > 0 then
	        print("Your score is better than 0")
	    else
	        print("My God, your score turned out to be 0")
	    end --与上一示例代码不同的是，此处要添加一个end
	end
	```
	
2. while

	```
	local t = {1, 3, 5, 8, 11, 18, 21}
	
	local i
	for i, v in ipairs(t) do
	    if 11 == v then
	        print("index[" .. i .. "] have right value[11]")
	        break
	    end
	end
	```
	
	没有`continue`语句。

3. repeat

	```
	x = 10
	repeat
	    print(x)
	until false
	```
	
	死循环

4. for

	```
	for var = begin, finish, step do
	    --body
	end
	
	for i = 1, 5 do
	  print(i)
	end
	-->output 1 2 3 4 5
	
	for i = 1, 10, 2 do
	  print(i)
	end
	-->output 1 3 5 7 9
	
	-->如果不想给循环设置上限的话，可以使用常量 math.huge
	for i = 1, math.huge do
	    if (0.3*i^3 - 20*i^2 - 500 >=0) then
	      print(i)
	      break
	    end
	end
	
	local a = {"a", "b", "c", "d"}
	for i, v in ipairs(a) do
	  print("index:", i, " value:", v)
	end
	```
	
	泛型 for 循环与数字型 for 循环有两个相同点： （1）循环变量是循环体的局部变量； （2）决不应该对循环变量作任何赋值。

5. break,return

## 函数
### 函数定义
由于函数定义本质上就是变量赋值，而变量的定义总是应放置在变量使用之前，所以函数的定义也需要放置在函数调用之前。
如果参数列表为空，必须使用 () 表明是函数调用。
由于函数定义等价于变量赋值，我们也可以把函数名替换为某个 Lua 表的某个字段，例如

```
function foo.bar(a, b, c)
    -- body ...
end
```

此时我们是把一个函数类型的值赋给了 foo 表的 bar 字段。换言之，上面的定义等价于

```
foo.bar = function (a, b, c)
    print(a, b, c)
end
```

对于此种形式的函数定义，不能再使用 local 修饰符了，因为不存在定义新的局部变量了。

### 函数参数
Lua 函数的参数大部分是按值传递的。在调用函数的时候，若形参个数和实参个数不同时，Lua 会自动调整实参个数。调整规则：若实参个数大于形参个数，从左向右，多余的实参被忽略；若实参个数小于形参个数，从左向右，没有被实参初始化的形参会被初始化为 nil。
 Lua 还支持变长参数。若形参为 ... ,示该函数可以接收不同长度的参数。访问参数的时候也要使用 ... 。
示例代码：

```
local function func( ... )                -- 形参为 ... ,表示函数采用变长参数

   local temp = {...}                     -- 访问的时候也要使用 ...
   local ans = table.concat(temp, " ")    -- 使用 table.concat 库函数对数
                                          -- 组内容使用 " " 拼接成字符串。
   print(ans)
end

func(1, 2)        -- 传递了两个参数
func(1, 2, 3, 4)  -- 传递了四个参数

-->output
1 2
1 2 3 4
```

值得一提的是，LuaJIT 2 尚不能 JIT 编译这种变长参数的用法，只能解释执行。所以对性能敏感的代码，应当避免使用此种形式。

当函数参数是 table 类型时，传递进来的是 实际参数的引用，此时在函数内部对该 table 所做的修改，会直接对调用者所传递的实际参数生效。
在常用基本类型中，除了 table 是按址传递类型外，其它的都是按值传递参数。 用全局变量来代替函数参数的不好编程习惯应该被抵制，良好的编程习惯应该是减少全局变量的使用。

### 函数返回值
允许函数返回多个值,返回多个值时，值之间用 “,” 隔开。

```
local function swap(a, b)   -- 定义函数 swap，实现两个变量交换值
   return b, a              -- 按相反顺序返回变量的值
end

local x = 1
local y = 20
x, y = swap(x, y)           -- 调用 swap 函数
print(x, y)                 --> output   20     1
```

当函数返回值的个数和接收返回值的变量的个数不一致时，Lua 也会自动调整参数个数。
调整规则： 若返回值个数大于接收变量的个数，多余的返回值会被忽略掉； 若返回值个数小于参数个数，从左向右，没有被返回值初始化的变量会被初始化为 nil。

当一个函数有一个以上返回值，且函数调用不是一个列表表达式的最后一个元素，那么函数调用只会产生一个返回值,也就是第一个返回值。
```
local function init()       -- init 函数 返回两个值 1 和 "lua"
    return 1, "lua"
end

local x, y, z = init(), 2   -- init 函数的位置不在最后，此时只返回 1
print(x, y, z)              -->output  1  2  nil

local a, b, c = 2, init()   -- init 函数的位置在最后，此时返回 1 和 "lua"
print(a, b, c)              -->output  2  1  lua
```
函数调用的实参列表也是一个列表表达式。考虑下面的例子：
```
local function init()
    return 1, "lua"
end

print(init(), 2)   -->output  1  2
print(2, init())   -->output  2  1  lua
```
如果你确保只取函数返回值的第一个值，可以使用括号运算符，例如
```
local function init()
    return 1, "lua"
end

print((init()), 2)   -->output  1  2
print(2, (init()))   -->output  2  1
```
值得一提的是，如果实参列表中某个函数会返回多个值，同时调用者又没有显式地使用括号运算符来筛选和过滤，则这样的表达式是不能被 LuaJIT 2 所 JIT 编译的，而只能被解释执行。

### 全动态函数调用

调用回调函数，并把一个数组参数作为回调函数的参数。
```
local args = {...} or {}
method_name(unpack(args, 1, table.maxn(args)))
```
使用场景
如果你的实参 table 中确定没有 nil 空洞，则可以简化为`method_name(unpack(args))`
你要调用的函数参数是未知的；
函数的实际参数的类型和数目也都是未知的。
伪代码
```
add_task(end_time, callback, params)

if os.time() >= endTime then
    callback(unpack(params, 1, table.maxn(params)))
end
```
值得一提的是，unpack 内建函数还不能为 LuaJIT 所 JIT 编译，因此这种用法总是会被解释执行。对性能敏感的代码路径应避免这种用法。
```
local function run(x, y)
    print('run', x, y)
end

local function attack(targetId)
    print('targetId', targetId)
end

local function do_action(method, ...)
    local args = {...} or {}
    method(unpack(args, 1, table.maxn(args)))
end

do_action(run, 1, 2)         -- output: run 1 2
do_action(attack, 1111)      -- output: targetId    1111
```

## 模块
Lua 提供了一个名为 require 的函数用来加载模块。要加载一个模块，只需要简单地调用 require "file"就可以了，file 指模块所在的文件名。这个调用会返回一个由模块函数组成的 table ，并且还会定义一个包含该 table 的全局变量。+

在 Lua 中创建一个模块最简单的方法是：创建一个 table ，并将所有需要导出的函数放入其中，最后返回这个 table 就可以了。相当于将导出的函数作为 table 的一个字段，在 Lua 中函数是第一类值，提供了天然的优势。
把下面的代码保存在文件 my.lua 中
```
local foo={}

local function getname()
    return "Lucy"
end

function foo.greeting()
    print("hello " .. getname())
end

return foo
```
把下面代码保存在文件 main.lua 中，然后执行 main.lua，调用上述模块。
```
local fp = require("my")
fp.greeting()     -->output: hello Lucy
```
注：对于需要导出给外部使用的公共模块，处于安全考虑，是要避免全局变量的出现。我们可以使用 lua-releng 工具完成全局变量的检测，具体参考 lua 的 局部变量 章节。

## string库
Lua 字符串总是由字节构成的。Lua 核心并不尝试理解具体的字符集编码（比如 GBK 和 UTF-8 这样的多字节字符编码）。
Lua 字符串内部用来标识各个组成字节的下标是从 1 开始的，string.sub(str, 3, 7) 直接表示从第三个字符开始到第七个字符（含）为止的子串。

string.byte(s[,i[,j]]) 返回ASCII码，i,j默认为1

string.char(...)       返回0个或多个整数(0~255)对应的ASCII码组成的字符串

string.upper(s)        返回把所有字符转成大写的字符串

string.lower(s)        返回把所有字符转成小写的字符串

string.len(s)          返回长度。应当使用`#`运算符来获取Lua字符串长度

string.find(s,p[,init[,plain]]) 在字符串 s 中匹配（模式）字符串 p，若匹配成功，则返回目标字符串中与模式匹配的子串；否则返回 nil。

string.format(formatstring,...) 格式化，与C类似 `print(string.format("%.4f",3.1415926))`保留四位小数

string.match(s,p[,init])

string.gmatch(s,p) 返回一个迭代器函数，通过这个迭代器函数可以遍历到在字符串s中出现模式串p的所有地方。

string.rep(s,n)  返回字符串s的n次拷贝

string.sub(s,i[,j]) 返回字符串 s 中，索引 i 到索引 j 之间的子字符串。

string.gsub(s,p,r[,n]) 将目标字符串 s 中所有的子串 p 替换成字符串 r。可选参数 n，表示限制替换次数。返回值有两个，第一个是被替换后的字符串，第二个是替换了多少次。

string.reverse(s)  接收一个字符串s，返回这个字符串的反转。

## table库
在 Lua 中，数组下标从 1 开始计数。在初始化一个数组的时候，若不显式地用键值对方式赋值，则会默认用数字作为下标，从 1 开始。由于在 Lua 内部实际采用哈希表和数组分别保存键值对、普通值，所以不推荐混合使用这两种赋值方式。

table.getn(table) 获取长度，取长度的一元操作#

table.concat(table[,sep[,i[,j]]]) 拼接

table.insert(table,[pos,]value)

table.maxn(table) 返回（数组型）表 table 的最大索引编号；如果此表没有正的索引编号，返回 0。

table.remove(table[,pos]) 在表 table 中删除索引为 pos（pos 只能是 number 型）的元素，并返回这个被删除的元素，它后面所有元素的索引值都会减一。pos 的默认值是表的长度，即默认是删除表的最后一个元素。

table.sort(table[,comp]) 

table.new

table.clear

## 日期时间函数
Lua: time,date,difftime
```
os.time([table])
os.difftime(t2,t1)
os.date([format[,time]])
```

OpenResty: ngx_lua: ngx.today,ngx_time,ngx_utction,ngx_localtime,ngx.now,ngx.http_time,ngx_cookie_time

## 数学库
```
math.rad(x) 角度转弧度
math.deg(x) 弧度转角度
math.max(x,...)
math.min(x,...)
math.random([m[,n]])
math.randomseed(x)
math.abs(x)
math.fmod(x,y) x对y取余数
math.pow(x,y)
math.sqrt(x)
math.exp(x) e的x次方
math.log(x)
math.log10(x)
math.floor(x)
math.ceil(x)
math.pi
math.sin(x) 弧度
math.cos(x)
math.tan(x)
math.asin(x)
math.acos(x)
math.atan(x)
```

## 文件操作
在OpenResty中，文件IO操作会产生阻塞效应。
- 隐式文件描述
	设置一个默认的输入或输出文件，然后在这个文件上进行所有的输入或输出操作。所有的操作函数由 io 表提供。
	```
	file = io.open("test1.txt", "a+")   -- 使用 io.open() 函数，以添加模式打开文件
	io.output(file)                     -- 使用 io.output() 函数，设置默认输出文件
	io.write("\nhello world")           -- 使用 io.write() 函数，把内容写到文件
	io.close(file)
	```
- 显示文件描述
	使用 file:XXX() 函数方式进行操作,其中 file 为 io.open() 返回的文件句柄。
	```
	file = io.open("test2.txt", "a")  -- 使用 io.open() 函数，以添加模式打开文件
	file:write("\nhello world")       -- 使用 file:open() 函数，在文件末尾追加内容
	file:close()
	```

文件操作函数
```
io.open (filename [, mode])
file:close ()
io.close ([file])
file:flush ()
io.flush ()
io.input ([file])
file:lines ()
io.lines ([filename])
io.output ([file])
file:read (...)
io.read (...)
io.type (obj)
file:write (...)
io.write (...)
file:seek ([whence] [, offset])
file:setvbuf (mode [, size])
```

## FFI
FFI 库，是 LuaJIT 中最重要的一个扩展库。它允许从纯 Lua 代码调用外部C函数，使用 C 数据结构。有了它，就不用再像 Lua 标准 math 库一样，编写 Lua 扩展库。把开发者从开发 Lua 扩展 C 库（语言/功能绑定库）的繁重工作中释放出来。
```
local ffi = require("ffi")
ffi.cdef[[
int printf(const char *fmt, ...);
]]
ffi.C.printf("Hello %s!", "world")
```

```
local ffi = require("ffi")
ffi.cdef[[
unsigned long compressBound(unsigned long sourceLen);
int compress2(uint8_t *dest, unsigned long *destLen,
        const uint8_t *source, unsigned long sourceLen, int level);
int uncompress(uint8_t *dest, unsigned long *destLen,
         const uint8_t *source, unsigned long sourceLen);
]]
local zlib = ffi.load(ffi.os == "Windows" and "zlib1" or "z")

local function compress(txt)
  local n = zlib.compressBound(#txt)
  local buf = ffi.new("uint8_t[?]", n)
  local buflen = ffi.new("unsigned long[1]", n)
  local res = zlib.compress2(buf, buflen, txt, #txt, 9)
  assert(res == 0)
  return ffi.string(buf, buflen[0])
end

local function uncompress(comp, n)
  local buf = ffi.new("uint8_t[?]", n)
  local buflen = ffi.new("unsigned long[1]", n)
  local res = zlib.uncompress(buf, buflen, comp, #comp)
  assert(res == 0)
  return ffi.string(buf, buflen[0])
end

-- Simple test code.
local txt = string.rep("abcd", 1000)
print("Uncompressed size: ", #txt)
local c = compress(txt)
print("Compressed size: ", #c)
local txt2 = uncompress(c, #txt)
assert(txt2 == txt)
```
[?]意味着他是一个变长数组

下面是一个使用 C 数据结构的实例
```
local ffi = require("ffi")
ffi.cdef[[
typedef struct { double x, y; } point_t;
]]

local point
local mt = {
  __add = function(a, b) return point(a.x+b.x, a.y+b.y) end,
  __len = function(a) return math.sqrt(a.x*a.x + a.y*a.y) end,
  __index = {
    area = function(a) return a.x*a.x + a.y*a.y end,
  },
}
point = ffi.metatype("point_t", mt)

local a = point(3, 4)
print(a.x, a.y)  --> 3  4
print(#a)        --> 5
print(a:area())  --> 25
local b = a + point(0.5, 8)
print(#b)        --> 12.5
```

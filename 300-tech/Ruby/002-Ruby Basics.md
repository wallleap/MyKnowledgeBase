---
title: 002-Ruby Basics
date: 2024-07-15 15:52
updated: 2024-07-16 10:59
---

## 安装

推荐使用 Ruby 版本管理工具 RVM（Ruby Version Manager） 进行安装

Linux(Ubuntu)

```sh
sudo apt-get update
sudo apt-get install curl
curl -L https://get.rvm.io | bash -s stable
rvm install 3.0.0
```

macOS

```sh
brew install curl
curl -L https://get.rvm.io | bash -s stable
rvm install 3.0.0
```

Windows 上推荐使用 Docker + VSCode 的 Remote 插件在 Linux 环境下使用 Ruby

RVM 基础命令

```sh
rvm list know # 列出可安装的 Ruby 版本
rvm list # 已安装的版本
rvm use 3.0.0 # 切换版本为
rvm use --default 3.0.0 # 设置为系统默认版本
rvm uninstall 2.2.1 # 卸载指定版本
```

## 运行 Ruby 程序

例如 file.rb

```rb
puts "You gave #{ARGV.size} arguments"
p ARGV
```

运行

```sh
ruby file.rb apple pear banana
```

Command Line Tools 命令行工具

- irb：Interactive Ruby
- rdoc：Ruby Documentation（ri）

```sh
$ irb
3.0.0 :001 > a = 1
 => 1
3.0.0 :002 > a
 => 1
3.0.0 :003 > a = "hello"
 => "hello"
3.0.0 :004 > a
 => "hello"
3.0.0 :005 > def hi
3.0.0 :006 >   'hi'
3.0.0 :007 > end
 => :hi
3.0.0 :008 > hi
 => "hi"
3.0.0 :009 > a.length
3.0.0 :009 > a.length
 => 5
```

Other Documentation Tools

- <http://apidock.com/ruby>
- <http://ruby-doc.org/core-2.3.0/>
- <http://api.rubyonrails.org/>
- Dash: <https://kapeli.com/dash>（文档软件）

## 基础数据类型

```rb
# Fixnum
3
343
# Float
3.43
# String
hello
hello world
# Boolean
true(TrueClass)
false(FalseClass)
# Array
[1, 2]
["hello", "hello world"]
# Hash
{"name"=>"Tom", "age"=>18}
{name: "tom", age: 18}
# Symbol
:a
:hello
:"hello world"
# Range
1..10
1...10 # 不包含10，输出1-9
# Regular Expression
/hello/i
```

例如

```sh
$ irb
3.0.0 :011 > a = 3.14
 => 3.14
3.0.0 :013 > a
 => 3.14
3.0.0 :014 > a.class
 => Float
3.0.0 :015 > a = 1
 => 1
3.0.0 :016 > a.class
 => Integer
3.0.0 :019 >
3.0.0 :020 > s = "hello"
 => "hello"
3.0.0 :021 > s.class
 => String
3.0.0 :023 > s1 = "hello"
 => "hello"
3.0.0 :024 > s2 = "world"
 => "world"
3.0.0 :025 > s1 + " " + s2
 => "hello world"
3.0.0 :026 > "#{s1} #{s2}"
 => "hello world"
3.0.0 :027 > s.length
 => 5
3.0.0 :028 > "hello world".capitalize
 => "Hello world"
3.0.0 :029 > "hello world".capitalize!
 => "Hello world"
3.0.0 :030 > "hello world".gsub("world", "Tom").upcase
 => "HELLO TOM"
```

## 变量

变量赋值

```rb
3.0.0 :035 > a = "hello"
 => "hello"
3.0.0 :036 > a.object_id
 => 18780
3.0.0 :037 > a.replace("hello2")
 => "hello2"
3.0.0 :038 > a.object_id
 => 18780 # 改变内存地址中的内容
3.0.0 :039 > a = "hello"
 => "hello"
3.0.0 :040 > a.object_id
 => 18800 # 进行赋值了，就改变了内存地址

3.0.0 :045 > a = ["a", "b"]
 => ["a", "b"]
3.0.0 :046 > a.length
 => 2
3.0.0 :047 > a[0]
 => "a"
3.0.0 :048 > a[1]
 => "b"
3.0.0 :049 > a[2]
 => nil

3.0.0 :051 > a = {"a": 1, "b": 2}
 => {:a=>1, :b=>2}
3.0.0 :052 > a.length
 => 2
3.0.0 :053 > a[:b]
 => 2
3.0.0 :054 > a[:a]
 => 1
3.0.0 :055 > a = {"a" => 1, :b => 2}
 => {"a"=>1, :b=>2}
3.0.0 :056 > a["a"]
 => 1
3.0.0 :057 > a[:b]
 => 2
3.0.0 :058 > a["b"]
 => nil
```

调用方法：`xxx.func`

以 `!` 结尾的方法：会改变变量自身（这只是一个约束/约定）；在 Rails 中也用来表示该方法会抛出异常

```rb
3.0.0 :071 > a = "hello world"
 => "hello world"
3.0.0 :061 > a.capitalize
 => "Hello world"
3.0.0 :073 > a
 => "hello world"
3.0.0 :074 > a.capitalize!
 => "Hello world"
3.0.0 :075 > a
 => "Hello world"
```

以 `?` 结尾的方法：会返回 `true|false`（这只是一个约束）

```rb
3.0.0 :076 > a = "hello"
 => "hello"
3.0.0 :077 > a.is_a?(String)
 => true
```

`nil` 为空

```rb
3.0.0 :080 > a = nil
 => nil
3.0.0 :081 > a.nil?
 => true
# 在 Ruby 中 nil 和 false 都是不成立/否的意思，其他一切都是 true
```

双引号和单引号：双引号中可以放变量解释

```rb
3.0.0 :094 > a = "hello"
 => "hello"
3.0.0 :095 > b = 'hello'
 => "hello"
3.0.0 :096 > a == b
 => true
3.0.0 :097 > c = "world"
 => "world"
3.0.0 :098 > a = "hello #{c}"
 => "hello world"
```

反引号中放 Shell 命令

```rb
`ls`
```

## Array 和 Hash

```rb
3.0.0 :105 > a = [1, 2, "hello"]
 => [1, 2, "hello"]
3.0.0 :106 > a.length
 => 3
3.0.0 :107 > a.size
 => 3
3.0.0 :108 > a.first
 => 1
3.0.0 :109 > a.last
 => "hello"
3.0.0 :110 > b = ["world"]
 => ["world"]
3.0.0 :111 > c = a + b
 => [1, 2, "hello", "world"]
3.0.0 :112 > b * 3
 => ["world", "world", "world"]
3.0.0 :113 > b
 => ["world"]
3.0.0 :114 > c - a
 => ["world"]
```

上面这些方法并不会改变数组本身

```rb
3.0.0 :126 > a = []
 => []
3.0.0 :127 > a.push(1)
 => [1]
3.0.0 :128 > a.push(2)
 => [1, 2]
3.0.0 :129 > a.unshift(3)
 => [3, 1, 2]
3.0.0 :130 > a
 => [3, 1, 2]
3.0.0 :131 > a.pop
 => 2
3.0.0 :132 > a.shift
 => 3
3.0.0 :133 > a
 => [1]

3.0.0 :134 > a = []
 => []
3.0.0 :135 > a << 3
 => [3]
3.0.0 :136 > a
 => [3]
3.0.0 :137 > a.concat([4, 5])
 => [3, 4, 5]
3.0.0 :138 > a.index(4)
 => 1
3.0.0 :139 > a[0] = 1
 => 1
```

上面这些方法会改变数组本身

```rb
3.0.0 :140 > a.max # 数组中最大值
 => 5
```

Ruby 中的方法，可以这样理解 `a = 1;a+2#=>3` 把 `+` 等操作符看作方法，`2` 是传递的参数

方法的定义和参数传递都可以不使用 `()`，例如 `a = []; a.push 1; a.push(2)`

Hash 是无序的（key-value 结构），数组是有序的

```rb
3.0.0 :145 > a = { "name" => "Tom", "age" => 18, "hobbies" => ["game", "music"] }
 => {"name"=>"Tom", "age"=>18, "hobbies"=>["game", "music"]}
3.0.0 :146 > a["name"]
 => "Tom"
3.0.0 :151 > a.keys
 => ["name", "age", "hobbies"]
3.0.0 :152 > a.values
 => ["Tom", 18, ["game", "music"]]
3.0.0 :153 > a.delete("age")
 => 18
3.0.0 :154 > a["phone"] = 10086
 => 10086
3.0.0 :155 > a
 => {"name"=>"Tom", "hobbies"=>["game", "music"], "phone"=>10086}
```

## Symbol

Symbol 是 String 的补充，可以看作字符串来使用，但是他们在本质上是不同的，在 Ruby 中 Symbol 经常被用来作为 Hash 的 key 和一些变化不频繁的字符串

```rb
3.0.0 :159 > a = :hello
 => :hello
3.0.0 :160 > b = "#{a} world"
 => "hello world"
3.0.0 :161 >
3.0.0 :162 > c = :"hello world"
 => :"hello world"
3.0.0 :163 > c.object_id
 => 3167988
3.0.0 :164 > c = :"hello world"
 => :"hello world"
3.0.0 :165 > c.object_id # 不变
 => 3167988
3.0.0 :168 >
3.0.0 :169 > a = { :name => "Tom", :age => 18 }
 => {:name=>"Tom", :age=>18}
3.0.0 :170 > a = { "name": "Jack", "age": 20 } # 为了顺应前端发展趋势
 => {:name=>"Jack", :age=>20}
3.0.0 :171 > a = { name: "Jacky", age: 18 }
 => {:name=>"Jacky", :age=>18}
```

## Range

```rb
3.0.0 :169 > a = { :name => "Tom", :age => 18 }
 => {:name=>"Tom", :age=>18}
3.0.0 :170 > a = { "name": "Jack", "age": 20 }
 => {:name=>"Jack", :age=>20}
3.0.0 :171 > a = { name: "Jacky", age: 18 }
 => {:name=>"Jacky", :age=>18}
3.0.0 :172 >
3.0.0 :173 >
3.0.0 :174 > a = 1..10
 => 1..10
3.0.0 :175 > a.to_a.size
 => 10
3.0.0 :176 > b = 1...10
 => 1...10
3.0.0 :177 > b.to_a.size
 => 9
3.0.0 :178 > c = :a..:z
 => :a..:z
3.0.0 :179 > d = a.to_a + c.to_a
 =>
[1,
...
:z]
```

## Regular Expression

```rb
3.0.0 :186 > a = /hello/
 => /hello/
3.0.0 :187 > "hello world" =~a
 => 0
3.0.0 :188 > a = /hello/
 => /hello/
3.0.0 :189 > "hello world" =~ a
 => 0
3.0.0 :190 > "Hello, hello" =~ a
 => 7
3.0.0 :193 > email_reg = /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
 => /\A[\w+\-.]+@[a-z\d\-.]+\.[a-z]+\z/i
3.0.0 :194 > email_reg.match("111@qq.com")
 => #<MatchData "111@qq.com">
```

## Method 方法

### 参数

直接定义，传参调用，括号可省略

```rb
def hi(name)
  p "hi " + name
end
hi("Tom") # "hi Tom"
hi "Jack" # "hi Jack"

def hello name
  p "hello #{name}"
end
hello("123") # "hello 123"
```

设置默认参数

```rb
def hi name="Ruby"
  p "hi " + name
end
hi # "hi Ruby"
hi "Tom" # "hi Tom"

def hi name, age=18 # 同名会覆盖上面的方法
  p "#{name} is #{age}"
end
hi "Jack" # "Jack is 18"
hi "Tom", 18 # "Tom is 18"
```

### 返回值

所有方法都有返回值，方法体**最后一行代码的返回值**默认会作为方法的返回值，也可以显式地使用 `return` 关键字

```rb
def hi
  p "OK"
end
p hi # => "OK" "OK"

def hi
  return '2'
  p '1' # return 后面的代码不会执行
end
p hi # => "2"
```

## Block 代码块

类似于一个方法，可以使用 `do end` 来定义，也可以使用 `{}`

```rb
File.open('access.log', 'w+') do |f| # 接收的参数用管道符包裹
  f.puts 'line 1'
end
```

Block 内置方法

```rb
[1, 2, 3].each do |x|
  p x
end
# => 1 2 3
[1, 2, 3].each { |x|
  p x
}

{a: 1, b: 2}.each do |x, y|
  p y
end
# => 1 2
```

Block 变量作用域

```rb
# Ruby 2.0 之前外部变量 x 的值会被 block 修改；现在版本已经不会
x = 10
[1, 2, 3].each do |x|
  p x
end
p x # => 10
```

自定义 Block

- `yield` 为调用外部 block 的关键字
- `proc` 相当于定义了一个方法变量，可以把方法当作参数传递
- `lambda` 和 `proc` 非常相似

```rb
def hi name
  yield(name)
end
hi('code') { |x|
  p "hello #{x}"
}
# => "hello code"

def hi
  yield
  yield
end
hi {
  p 'ok'
}
# => 'ok' 'ok'


hi = proc { |x|
  p "hello #{x}"
}
hi.call('world') # => 'hello world'
# 相当于
hi = Proc.new { |x|
  p "hello #{x}"
}
hi.call('world')


hi = lambda { |x|
  p "hello #{x}"
}
hi.call('world')
```

## 条件语句

Control Structures

if-elsif-else：只有 `false` 和 `nil` 是否

```rb
a = 'hello'
b = false
if a
  p a
elsif b
  p b
else
  p 'ok'
end
# => 'hello'
```

unless：相当于 if 的反向断言

```rb
unless false
  p 'ok'
end
# => 'ok'
if not false
  p 'yes'
end
# => 'yes'
```

单行赋值

```rb
a = 1 if a != 1
p a # => 1
b = 2 if not defined?(b)
p b # => nil
```

case

```rb
a = [1, 2, 3]
case a
when 1
  p 1
when /hello/
  p 'hello world'
when Array
  a.each { |x| p x }
else
  p 'ok'
end
# => 1 2 3
```

while

```rb
a = 0
while a < 3 do
  p a
  a += 1
end
# => 0 1 2
```

Ruby 没有 `++` 和 `--` 操作符

Iterators 迭代

```rb
for x in [1, 2, 3] do
  p x
end
# 1 2 3

[1, 2, 3].each do |x|
  p x
end
# 1 2 3

10.times { |t| p t } # 0~9
```

## 代码块和异常

### Blocks

- Block 是一个参数
- 匿名函数
- Callback
- 使用 `do end` 或者 `{}` 来定义

```rb
def hello
  puts 'hello method start'
  yield # 传入的执行代码块
  yield
  puts 'hello method end'
end
hello { puts 'i am in block' }
# hello method start
# i am in block
# i am in block
# hello method end
```

传入的代码块，可以传递参数

```rb
def hello
  puts 'hello method start'
  yield('hello', 'world')
  puts 'hello method end'
end
hello { |x, y| puts "i am in block, #{x} #{y}" }
# hello method start
# i am in block, hello world
# hello method end
```

函数传递参数，代码块能接着传

```rb
def hello name
  puts 'hello method start'
  result = "hello " + name
  yield(result)
  puts 'hello method end'
end
hello('Ruby') { |x| puts "I'm in block, I got #{x}" }
# hello method start
# I'm in block, I got hello Ruby
# hello method end
```

内置的一些方法，能使用代码块传递参数

```rb
['cat', 'dog', 'mouse'].each do |animal|
  puts animal
end

['cat', 'dog', 'mouse'].each { |animal| puts animal }

10.times do |t|
  puts t
end

('a'..'d').each { |char| puts char }
```

### 代码块中的变量作用域

```rb
x = 1
[1, 2, 3].each { |x| puts x }
puts x # 1

sum = 0
[1, 2, 3].each { |x| sum += x }
puts sum  # 6
# puts x # x is invalid
```

### 代码块返回值

```rb
class Array
  def find
    each do |value|
      return value if yield(value)
    end
    nil
  end
end
puts [1, 2, 3].find { |x| x == 2 } # 2
```

### 代码块作为命名参数

```rb
def hello name, &block
  puts "hello #{name}, from method"
  block.call(name)
end
hello("Ruby") { |x| puts "hello #{x} from block" }
# hello Ruby, from method
# hello Ruby from block
```

### 判断代码块是否被传递

```rb
def hello
  if block_given? # 内置方法
    yield
  else
    puts "hello from method"
  end
end
hello
hello { puts "hello from block" }
# hello from method
# hello from block
```

### 闭包

```rb
def counter
  sum = 0
  proc { |x| x = 1 unless x; sum += x }
end
c2 = counter # 返回值为代码块
puts c2.call(1) # 1
puts c2.call(2) # 3
puts c2.call(3) # 6
```

### 其他方法创建代码块

```rb
# 必须传参数
hello = -> (name) { "hello #{name}" }
puts hello.call("Ruby")

hello3 = lambda { |name| "hello #{name}" }
puts hello3.call("Ruby")

# 可以不传参数
hello2 = proc { |name| "hello #{name}" }
puts hello2.call
puts hello2.call("Ruby")
```

### 常见 Exception

- StandardError
- SyntaxError
- RuntimeError
- ArgumentError
- NameError
- etc.

### 处理异常

```rb
def hello name
  raise name # 主动抛出异常
end
hello # ArgumentError
hello('123') # RuntimeError
```

常把需要捕获异常的代码块放到 `begin end` 中间

```rb
def hello
  raise
end

begin
  hello
rescue RuntimeError # 直到错误类型
  puts 'got it'
end
```

通用

```rb
begin
  hello
rescue => e
  puts "catch exception with name: #{e.class}" # NameError
else
  puts 'ok'
ensure
  puts 'ensure'
end
```

## 变量操作

变量交换

```rb
a = 1
b = 2
b, a = a, b
p a, b # 2 1

x = [1, 2, 3]
y, z = x
p y, z # 1 2

x = [1, 2, 3]
y, *z = x # 其余所有
p y, z # 1 [2, 3]
```

数值

```rb
p 1 / 10 # 0
p 1 / 10.0 # 0.1 有一个是浮点数结果就是浮点数
```

字符串

```rb
a = "world"
b = %Q{
  hello
  #{a}
} # {} 可以换成 () 或 []
p b
# "\n  hello\n  world\n"

a = <<-HERE
hello world
#{b}
HERE
p a
# "hello world\n\n  hello\n  world\n\n"
```

数组

```rb
a = %w[a b c 1]
p a # ["a", "b", "c", "1"] 都是字符串
```

## 代码规范

- 使用 UTF-8 编码
- 使用空格缩进（2 空格）
- 不需要使用分号（`;`）和反斜杠（`\`）连接代码

```rb
# 运算符一般左右留一个空格
a = 1
b = "hello world"

# 单行和多行（多行不需要\连接）
c = ["pear", "cat", "dog"]
c2 = [
  "pear",
  "cat",
  "dog"
]

d = { name: "Tom", age: 18 }
d2 = {
  name: "TOM",
  age: 18
}

def hello(name, age = 18)
  puts "hello #{name}, my age is #{age}"
end
# 可以省略括号
def hello name, age = 18
  puts "hello #{name}, my age is #{age}"
end
hello "TOM", 20
```

## 变量类型

- 局部变量 local variables：`a=1,b="hello"` 小写字母开头，单词之间 `_` 连接
- 常量 constants：`Names=['john', 'alex']` 大写字母开头
- 全局变量 global variables：`$platform='mac'`
- 实例变量 instance variables：`@name="Ruby"`
- 类变量 class variables：`@@counter=20`

常量不应该进行修改，直接重新赋值会有警告

```rb
Name = 'Ruby'
Name = 'Rails'
p Name
# warning: already initialized constant Name
# warning: previous definition of Name was here
# "Rails"

Name.replace 'Python'
p Name
# "Python"
```

类

```rb
class User
  attr_reader :name
  @@counter = 0
  def initialize name
    @name = name
    @@counter += 1
  end
  def get_counter
    @@counter
  end
end
user = User.new 'Tom'
p user.name # "Tom"
p user.get_counter # 1
user1 = User.new 'Jack'
p user.get_counter # 2
```

全局变量

```rb
def hello
  puts $$ # 内置全局变量 进程id
  p $: # 内置全局变量 Ruby 环境加载路径
end
hello
```

## 布尔表达式

`&&`、`||`、`!`、`and`、`or`、`not`

and 需要两个都为真才为真，只要有一个为假就假，前一个为假就不再判断后一个

or 只要有一个为真就真，两个都为假才假，前一个为真就不再运行后面

```rb
puts (true and true)
puts (true and false)
true and (puts "真后还运行")
puts (false and (puts "假后不运行"))
puts (true or false)
false or (puts "假后还运行")
puts (true or (puts "真后不运行"))
puts (not true)
```

关键字比传统符号（`&&`、`||`、`!`、= 等）的优先级要低

```rb
a = (false and false || true) # false 先 ||
b = (false and false or true) # true 顺序
```

方便赋值

```rb
a = nil
b = a || 4 # 4 为兜底值
p b # 4

c = b && 5 # 
p c # 5

b = a or 4
p b # nil
c = b && 5 # 
p c # nil
```

## String Hash Array 常用方法

字符串

```rb
a = "hello word"
p a.empty? # false

a[0] = "H"
p a # "Hello word"

a.freeze
# a[0] = "h" # an't modify frozen String FrozenError

a = "hello"
p a # "hello"

b = "hello world"
p b.reverse # "dlrow olleh"

p b.sub('o', 'A') # "hellA world"
p b.gsub('o', 'A') # "hellA wArld"

p b.start_with? 'h'
p b.end_with? 'd'
p b.include? 'o'

c = b.split(' ')
p c # ["hello", "world"]

p c.join(' ') # "hello world"
```

引用赋值

```rb
a = "hello world"
b = a
p b # "hello world"

b[0]= 'H'
p a # "Hello world"
```

使用一个方法

```rb
a = "hello world"
b = a.dup
p b # "hello world"

b[0]= 'H'
p b # "Hello world"
p a # "hello world"
```

`String#clone`

数组

```rb
a = ['pear', 'cat', 'horse']
p a.join(' ') # "pear cat horse"

# a.clear
# p a # []

a.find { |x| p x == 'horse' } # false false true

a.map { |x| p x.upcase } 
a.collect { |x| p x.upcase } # 两个作用一样

p a.uniq # 删除重复元素
p a.flatten # 多维转一维
p a.sort # 排序
p a.count # 数量

a.delete_if { |x| x == 'horse' } # 删除满足条件的元素
p a # ["pear", "cat"]

a.each { |x| p x } # "pear" "cat" 遍历
```

Hash

```rb
a = { name: "Tom", age: 18 }

a.each { |key, value| p key } # :name :age
p a.keys # [:name, :age]
p a.values # ["Tom", 18]
p a.has_key? :name # true
p a.delete :name # "Tom"
p a # {:age=>18}
p a.delete_if { |key| key == :age } # {}
p a # {}
```

其他

内置方法

- `send`：调用私有方法；方法作为变量
- `respond_to?`：

```rb
def hello
  p 'hello world'
end
send(:hello) # 方法作为变量（例如字符串拼接得到的方法名）

a = "hello world"
p a.respond_to?(:length) # 是否有length方法
```

## 代码加载机制

全局变量 `$LOAD_PATH`，在这些路径中查找

命名约束：文件名字和模块/类代码应该对应

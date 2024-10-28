---
title: Eval 家族的那些事儿
date: 2024-08-16T11:26:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
source: //ruby-china.org/topics/38329
---

许多编程语言都会附带 [eval](https://en.wikipedia.org/wiki/Eval) 的功能，通常会出现在动态语言中，它就有点像是一个微型的解释器，可以在运行时解释代码片段。这篇文章主要以 Ruby 为例，详细介绍 Ruby 中的 `eval` 家族。

## 代码片段的执行者 eval

Eval 是 Ruby 语言中比较有意思的一个功能了。其实不仅仅是 Ruby，许多语言都开放了这个功能。不过在不同语言里，该功能的命名方式以及侧重点会有所不同。

在 Lua 编程语言中，eval 的功能通过一个叫 `load`(版本 5.2 之后) 的函数来实现，不过它解释完代码片段之后会返回一个新的函数，我们需要手动调用这个函数来执行对应的代码片段

```
> load("print('Hello World!')")()
Hello World!
```

诡异的地方在于，它不能解析单纯的算术运算

```
> 1 + 2
3

> load("1 + 2")()
stdin:1: attempt to call a nil value
stack traceback:
    stdin:1: in main chunk
    [C]: in ?
```

要解析算术运算，需要把它们包装成方法体

```
> f = load("return 1 + 2")
> f()
3
```

在 Python 中该功能是通过名为 `eval` 的函数来实现，用起来就像是一个简单的 REPL

```
In [2]: eval
Out[2]: <function eval>

In [3]: eval('1 + 2')
Out[3]: 3

In [4]: eval('hex(3)')
Out[4]: '0x3'
```

不过奇怪的地方在于它不能直接解析 Python 中语句，比如说 `print` 语句

```
In [5]: eval('print(1 + 2)')
  File "<string>", line 1
    print(1 + 2)
        ^
SyntaxError: invalid syntax
```

要打印东西，可以考虑把上述语句封装成一个方法

```
In [12]: def c_print(name):
   ....:     print(name)
   ....:

In [13]: eval("c_print(1 + 2)")
3
```

相比之下，Ruby 的 eval 似乎就**没节操**得多，或许是因为借鉴了 Lisp 吧？它几乎能执行任何代码片段

```
> eval('print("hello")')
hello => nil
> eval('1 + 2')
 => 3
```

接下来我尝试用它来执行脚本文件中的代码片段，假设我有这样一个 Ruby 脚本文件

```
// example.rb
a = 1 + 2 + 3

puts a
```

想要执行这个文件，最直接的方式就是

```
ruby example.rb

6
```

然而你还可以通过 `eval` 来做这个事情

```
> content = File.read("example.rb") # 读取文件中的代码片段
 => "a = 1 + 2 + 3\n\nputs a\n"
> eval(content)
6
 => nil
```

当然 Ruby 中的 `eval` 绝不仅如此，且容我慢慢道来。

## Eval 与上下文

在 Ruby 中用 `eval` 来执行代码片段的时候会默认采用当前的上下文

```
> a = 10000
=> 10000
> eval('a + 1')
=> 10001
```

我们也可以手动传入当前上下文的信息，故而，以下的写法是等价的

```
eval('a + 1', binding)
=> 10001
```

`binding` 与 `eval` 在当前的作用域中都是私有方法

```
> self
 => main
> self.private_methods.grep(:binding)
=> [:binding]
> self.private_methods.grep(:eval)
=> [:eval]
```

在功能上，它们分别来自于 [Kernel#binding](https://ruby-doc.org/core-2.6.2/Kernel.html#method-i-binding) 与 [Kernel#eval](https://ruby-doc.org/core-2.6.2/Kernel.html#method-i-eval)。

```
> Kernel.singleton_class.instance_methods(false).grep(:eval)
 => [:eval]
> Kernel.singleton_class.instance_methods(false).grep(:binding)
=> [:binding]

> Kernel.eval('a + 1', Kernel.binding)
=> 10001
```

有了这两个东西，我们可以写出一些比较有意思的功能。

```
> def hello(a)
>   binding
> end

> ctx = hello('hello world')
```

```
> eval('print a') # 打印当前上下文的变量`a`
10000 => nil

> eval('print a', ctx) # 打印`hello`运行时上下文的变量`a`
hello world => nil
```

通过 `binding` 截取 `hello` 方法的上下文信息并存储在对象中，然后把该上下文传递至 `eval` 方法中。此外，上文的 `ctx` 对象其实也有它自己的 `eval` 方法，这是从 `Binding` 类中定义的实例方法。

```
> ctx.class
=> Binding

> Binding.instance_methods(false).grep /eval/
=> [:eval]
```

区别在于它是一个公有的实例方法，接收的参数也稍微有所 [不同](https://ruby-doc.org/core-2.6.2/Binding.html#method-i-eval)。更简单地我们可以用下面的代码去打印 `hello` 运行时上下文参数 `a` 的值。

```
ctx.eval('print a')
hello world => nil
```

## 更多的 `eval` 变种

在 Ruby 中 `eval` 其实还存在一些变种，比如我们常用的用于打开类/模块的方法 `class_eval`/`module_eval` 其实就相当于在类/模块的上下文中运行代码。为了在实例变量的上下文中运行代码，我们可以采用 `instance_eval`。

### a. class_eval/module_eval

在 Ruby 里面类和模块之间的关系密不可分，很多时候我们会简单地把模块看成是无法进行实例化的类，它们两本质上是差不多的。于是乎 `class_eval` 跟 `module_eval` 两个方法其实只是为了让编码更加清晰，两者功能上并无太大区别。

```
> class A; end
 => nil
> A.class_eval "def set_a; @a = 1000; end"
 => :set_a
> A.module_eval "def set_b; @b = 2000; end"
 => :set_b
> a = A.new
 => #<A:0x00007ff59d955fc0>
> a.set_a
 => 1000
> a.set_b
 => 2000
> a.instance_variable_get('@a')
 => 1000
> a.instance_variable_get('@b')
=> 2000
```

我们也可以通过多行字符串来定义相关的逻辑

```
> A.class_eval <<M
> def print_a
>  puts @a
> end
> M
 => :print_a

> a.print_a
1000
```

不过在正式编码环境中通过字符串来定义某些函数逻辑实在是比较蛋疼，毕竟这样的话就没办法受益于代码编辑器的高亮环境，代码可维护性也相对降低。语言设计者或许考虑到了这一点，于是我们可以以代码块的形式来传递相关的逻辑。等价写法如下

```
class A; end

A.class_eval do
  def set_a
    @a = 1000
  end

  def set_b
    @b = 2000
  end

  def print_a
    puts @a
  end
end

i = A.new
i.set_a
i.set_b

puts i
puts i.instance_variable_get('@a')
puts i.instance_variable_get('@b')
i.print_a
```

打印结果

```
#<A:0x00007fb75102cb18>
1000
2000
1000
```

与之前的例子所达成的效果是一致的，只不过写法不同。除此之外，他们两个的嵌套层级是不一样的

```
> A.class_eval do
>   Module.nesting
> end
 => []

> A.class_eval "Module.nesting"
 => [A]
```

实际上，我们还可以用最开始介绍的 `eval` 方法来实现相关的逻辑

```
> A.private_methods.grep(:binding)
 => [:binding]
```

可见对于类 `A` 而言，`binding` 是一个私有方法，因此我们可以通过动态发派来获取类 `A` 上下文的信息。

```
> class_a_ctx = A.send(:binding)
=> #<Binding:0x00007f98f910ae70>
```

拿到了上下文之后一切都好办了，可分别通过以下两种方式来定义类 `A` 的实例方法。

```
> class_a_ctx.eval 'def set_c; @c = 3000; end'
=> :set_c

> eval('def set_d; @d = 4000; end', class_a_ctx)
=> :set_d
```

简单验证一下结果

```
> a = A.new
=> #<A:0x00007f98f923c078>
> a.set_d
=> 4000
> a.set_c
=> 3000
> a.instance_variable_get('@d')
=> 4000
> a.instance_variable_get('@c')
=> 3000
```

### b. instance_eval

通过 `instance_eval` 可以在当前实例的上下文中运行代码片段，我们先简单地定义一个类 `B`

```
class B
  attr_accessor :a, :b, :c, :d, :e
  def initialize(a, b, c, d, e)
    @a = a
    @b = b
    @c = c
    @d = d
    @e = e
  end
end
```

实例化之后，分别用不同的方式来求得实例变量每个实例属性相加的值

```
> k = B.new(1, 2, 3, 4, 5)
 => #<B:0x00007f999fa2c480 @a=1, @b=2, @c=3, @d=4, @e=5>

> puts k.a + k.b + k.c + k.d + k.e
15

> k.instance_eval do
>   puts a + b + c + d + e
> end

15
```

这只是个简单的例子，在一些场景中还是比较有用的，比如可以用它来定义单例方法

```
> k.instance_eval do
>   def sum
>     @a + @b + @c + @d + @e
>   end
> end

> k.sum
=> 15

> B.methods.grep :sum
 => []
```

咱们依旧可以采用最原始的 `eval` 方法来实现类似的功能，这里暂不赘述。

## 安全问题

对于动态语言来说 `eval` 是一个很强大的功能，但随之也带来了不少的安全问题，最麻烦的莫过于代码注入了。假设你的代码可以用来接收用户输入

```
# string_handling.rb
def string_handling(method)
  code = "'hello world'.#{method}"
  puts "Evaluating: #{code}"
  eval code
end

loop { p string_handling (gets()) }
```

如果我们的用户都是善意用户的话，那没有什么问题。

```
> ruby string_handling.rb
slice(1)
Evaluating: 'hello world'.slice(1)
"e"

upcase
Evaluating: 'hello world'.upcase
"HELLO WORLD"
```

But，如果一个恶意的用户输入了下面的内容

```
slice(1); require 'fileutils'; FileUtils.rm_f(Dir.glob("*"))
```

那是不是有点好玩了？假设运行脚本的系统角色有足够的权限，那么当前目录下的所有东西都会被删除殆尽。利用动态发派来实现类似的功能或许更加安全一些

```
# string_handling_better.rb
def string_handling_better(method, *arg)
 'hello world'.send(method, *arg)
end
```

我们可以对用户的输入先进行预处理，然后再把它传递到定义好的 `string_handling_better` 方法中去。

```
> string_handling_better('slice', 1, 10)
 => "ello world"
```

## 尾声

这篇文章分别从不同的角度谈论了 `eval`，以及它的一些变种。它是一个很强大的功能，不过能力越大责任越大，相应的还会带来一定的风险，若使用不当会引发系统问题。现实编程生活当中，直接使用 `eval` 的场景并不多，毕竟代码写在字符串里面的话，少了编辑器的语法高亮还是会为程序员带来不少困扰。不过采用 `class_eval/module_eval` 来打开类/模块，并以代码块的方式来定制逻辑的案例倒是数见不鲜。

---

eval 这块可以扩展看看 DSL 风格配置的实现（比如 Puma 的配置文件），还有 AR AM 如何使用 eval 构建字段的 accessor 的

我最近在看 Rack 的配置实现，感觉后面可以写一些关于 DSL 的东西。

恩，AM、AR 那边是线程安全和代码安全（锁、冲突避免、隔离到独立模块），eval 是 Ruby 表达力的源泉啊！

---

自定义 Module:

```
module M; def m; self end; end
```

模块相当于类，你这样定义的话其实相当于定义实例方法，类/模块方法才会去单例类里面去找，所以如果你想要定义成类/模块方法则需要

```
module M
  def self.m
     self 
   end
end
```

---

你是指 `binding` 是 Kernel 内的一个类/模块方法吗？不好文档写的又是 Public Instance Method，帮有此问。

---

其实 Python 执行语句要用 `exec()`，`eval()` 只能运行一个表达式

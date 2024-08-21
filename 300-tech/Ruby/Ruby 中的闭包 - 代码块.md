---
title: Ruby 中的闭包 - 代码块
date: 2024-08-16T11:25:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/38385
---

在许多编程语言中都会有闭包这个概念。今天主要来谈谈 Ruby 中的闭包，它在这门语言中地位如何，以什么形式存在，主要用途有哪些？

## 闭包概要

维基百科里对闭包地解释如下

```
In programming languages, a closure, also lexical closure or function closure, is a technique for implementing lexically scoped name binding in a language with first-class functions. Operationally, a closure is a record storing a function together with an environment. The environment is a mapping associating each free variable of the function (variables that are used locally, but defined in an enclosing scope) with the value or reference to which the name was bound when the closure was created.
```

看起来很复杂是吧？其实我也看不太懂，建议英文不好的人还是学我去看 [中文版](https://zh.wikipedia.org/wiki/%E9%97%AD%E5%8C%85)。通俗来讲，闭包就是一个函数，它可以跟外部作用域打交道，访问并修改外部作用域的变量。在我们所熟悉的 JavaScript 这门语言中，所有的函数/方法都是闭包。

```
let a = 1;
function add_a() {
  return a += 1;
}

console.log(a)
console.log(add_a())
console.log(add_a())
console.log(add_a())
console.log(a)
```

结果如下

```
1
2
3
4
4
```

可见，函数 `add_a` 可以自由访问外部作用域的变量 `a`，并且能够在函数调用过程中持续维护着变量的值。这种特性为函数的 [柯里化](https://stackoverflow.com/questions/36314/what-is-currying) 带来了可能。

```
function add(a, b) {
 return a + b
}

console.log(add(1,2)) // => 3
```

柯里化之后可得

```
function add(a) {
  return function(b) {
    return a + b
  }
}

console.log(add(1)(2)) // => 3
```

得益于闭包的特性，上述两个函数虽然调用方式不同，不过它们所完成的工作是等价的，我们可以借此写出许多有趣的代码。然而闭包如果使用不当，或许会不小心修改了不应该修改的外部变量，特别是这些外部变量被多个程序单元共享的时候，可能会引发意想不到的系统问题。Ruby 在设计的时候也考虑到了这种问题，于是只要是通过 `def` 定义的方法，它都会创建一个封闭的词法作用域，在该作用域内不可访问外部的局部变量，外部信息只能够通过参数的形式传入到函数中。比如，下面这个代码片段是会报错的

```
a = 100

def add_a
  a = a + 1
end

puts add_a

```

```
Traceback (most recent call last):
    1: from a.rb:7:in `<main>'
a.rb:4:in `add_a': undefined local variable or method `a' for main:Object (NameError)
```

然而，如果没有闭包，Ruby 这门语言所能够提供的灵活性就很有限了。Matz 也考虑到了这点，Ruby 中并不是没有闭包，它只是以另一种方式来展现 -- 代码块。代码块也是 Ruby 元编程的重点内容，接下来我们以代码块的形式来重新定义 `add_a` 方法。

```
a = 100

define_method :add_a do
  a = a + 1
end

puts add_a # => 101
puts add_a # => 102
puts add_a # => 103
```

该例子中采用了 `define_method` 搭配代码块来定义方法，使得我们可以在函数体的中访问外部作用域的局部变量 `a`，使得该函数能够达到我们预期的效果。一个方法的定义，是否要形成封闭的作用域，不同的语言可能会有不同的权衡，Ruby 特地采用了代码块来表示闭包，有别于一般的方法定义，为这门语言增添了不少色彩。

PS: 当然形如 `@xxxx` 的实例变量便不受这种封闭作用域的制约。因为实例变量本身就是实例上下文共享的。

## 回调

在编程世界中，我们简单地称能够作为某个函数的参数，并且能够在该函数内部被调用的函数为回调函数。许多人听到回调函数就会想到回调地狱，然而个人觉得只要设计得当，并不是所有回调都会沦为**地狱**。在 Ruby 中几乎每一个通过 `def` 关键字定义地方法都默认接收一个代码块来作为回调函数，通常这个默认的 _ 回调函数参数 _ 并不需要显式声明，考虑以下代码片段

```
def print_message(message)
  yield(message) if block_given?
  puts 'The End!!'
end

print_message('Hello World') do |message|
  puts "I will print the message #{message}"
end
```

结果如下

```
I will print the message Hello World
The End!!
```

好玩吧，我们可以在调用方法的时候，在末尾以代码块的形式来定义回调逻辑。在被调用的方法的内部，通过 `block_given?` 来判断是否有代码块传入，如果有需要则通过关键字 `yield` 来运行对应的代码块，并传入相关的参数。这种以代码块作为回调的方式，为编码带来了一定的灵活性。然而或许在一些场景中这种隐式接收代码块的方式并不是那么直观，我们也可以显式地去声明这个参数。

```
def print_message(message, &block)
  block.call(message) if block_given?
  puts "The End!!"
end

print_message('Hello World') do |message|
  puts "I will print the message #{message}"
end
```

只是在这种场景中对应的参数 `&block` 会把我们传入的代码块转换成 `Proc` 对象，于是在这个例子中需要通过 `Proc#call` 方法来运行对应的代码块，而不再用 `yield` 关键字了。当然我们也可以直接往被调用的方法中传入一个 `Proc` 的对象

```
callback = Proc.new do |message|
  puts "I will print the message #{message}"
end

def print_message(message, &block)
  block.call(message) if block_given?
  puts "The End!!"
end

print_message('Hello World', &callback)
```

打印结果都是一样的

```
I will print the message Hello World
The End!!
```

以上两种代码块充当回调的方式，是 Ruby 中编码的常用手段。回调逻辑始终作为“最后一个参数”传入到被调用的方法中去，这或许也是一种**约定优于配置**的表现吧。

## 代码块的“意义”

刚开始接触 Ruby 的时候，我总觉得代码块是一个反人类的设计，明明就是一个闭包，为何要设计得这么异类。更奇怪的是，许多业内人士都觉得代码块是 Ruby 最伟大的发明之一。后来接触多了渐渐也就习惯了，代码块的优雅配合上其闭包的特性，再加上上面所说的一些回调的相关约定，为 Ruby 这门语言增色不少。代码块有两种表达方式 `{ .... }` 和 `do ... end`。一般对于单行的代码块会采用第一种形式，对于多行的代码块会采用第二种形式。在 Ruby 的开源世界中，代码块几乎无处不在，下面我们来看一些常见的案例

### 1. 容错设计

Ruby 承袭于 Lisp，代码块的运行会自动返回最后一条语句或者表达式的值，于是有些库也考虑到了用代码块来进行容错处理。就拿 `Hash` 的实例来作做个例子，我们希望当 `Hash` 实例对应的键值对不存在的时候给它一个默认值，常见的做法是

```
> hash = {}
> value = hash['a'] ? 'default value' : hash['a']
 => "default value"
```

熟悉 JavaScript 的人应该对上面这种代码不陌生，真所谓是啰嗦至极。为了使代码更加优雅我们可以采用 `Hash#fetch` 接口来取值，当 `Hash#fetch` 接口找不到对应的键值对的时候就会触发异常

```
> hash = {}
> hash.fetch('a')
Traceback (most recent call last):
        3: from /Users/lan/.rvm/rubies/ruby-2.5.3/bin/irb:11:in `<main>'
        2: from (irb):12
        1: from (irb):12:in `fetch'
KeyError (key not found: "a")
```

这个时候我们可以采用代码块来做容错，当找不到对应键的时候为取值操作提供一个默认值

```

> value = hash.fetch('a') { 'default value' }
 => "default value"
```

相对于第一种方式第二种方式更加优雅，也更有 Ruby 味一些。**虽说计算机世界是由 0，1 组成的，非此即彼。但是 Ruby 社区并不崇尚 Python 社区的绝对正确，每个人的偏向不同，我们可以选择自己喜欢的方式去完成工作。**

### 2. DSL

另外一个代码块用得比较广泛的地方应该就是 DSL 了，许多优秀的 Ruby 开源项目都会有相应的 DSL。下面是 Ruby 模板渲染库 [RABL](https://github.com/nesquena/rabl) 的配置代码

```
Rabl.configure do |config|
  # Enabling cache_all_output will cause an orders cache entry to be used in all templates
  # matching orders.cache_key, which results in unexpected behavior on Spree api response.
  # For more about this option, see https://github.com/nesquena/rabl/issues/281#issuecomment-6780104
  config.cache_all_output = false
  config.cache_sources = !Rails.env.development?
  config.view_paths = [Rails.root.join('app/views/')]
end
```

这是一个简单的 DSL，通过暴露模块内部的 `config` 实例，然后在调用者的上下文中去配置实例相关的属性。这里的代码块其实也充当了回调函数的角色，它让我们的配置逻辑可以被统一规划到一个区间当中，否则的话可能你得写出类似这样的配置代码

```
config = Rabl::ConfigureXXXX.new
config.cache_all_output = false
config.cache_sources = !Rails.env.development?
config.view_paths = [Rails.root.join('app/views/')]
```

无论怎么看都是 DSL 的方式比较优雅对吧？类似的 DSL 还有很多，这里不一一举例了，这些 DSL 如何去实现也不在本篇文章的讨论范围内。

### 3. 蹩脚的函数

代码块可以被看成是一个“蹩脚”的函数，虽说一般情况下它可以作为某个方法的回调，但是它不像 JavaScript 中的函数那样可以独立存在，它必须要依赖其他的机制。当我们要用代码块去定义一个匿名函数时，需要搭配 `lambda` 关键字或者 `Proc` 类来实现

```
> c = lambda() {}
 => #<Proc:0x00007ff47b8546a8@(irb):6 (lambda)>
> c.class
 => Proc

> (lambda() { 'hello' }).call
 => "hello"
> (lambda() { 'hello' })[]
 => "hello"
> (Proc.new { 'hello' }).call
 => "hello"
> (Proc.new { 'hello' })[]
 => "hello"
```

以上都是常用的定义匿名函数的方式，本质上它们都是 `Proc` 类的实例，需要显式地利用 `Proc#call` 方法或者语法糖 `[]` 来调用它们。

## 尾声

这篇文章简单地介绍了一下闭包的概念，闭包跟一般封闭作用域的方法有何不同之处。区别于一般的方法，闭包在 Ruby 中以代码块的形式出现，它在 Ruby 世界中几乎无处不在，充当了一等公民。这种区分，不仅使我们的 Ruby 代码更加优雅，增添了可读性，还使得我们的编码过程更加简单。

---

用 ruby2.3，使用 def 定义函数引用外部的变量，提示的并不是找不到变量，而是 undefined method '+' fro nil:NilClass。感觉是外部变量找不到了就那个 a 处理成了空类了吗。。

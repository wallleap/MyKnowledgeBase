---
title: 001-Ruby Introduction
date: 2024-07-15 14:36
updated: 2024-08-04 22:12
---

## 学习路线建议

**Ruby 的学习路线建议**

1. Ruby 高级开发，[https://eggman.tv/c/s-ruby-advanced](https://eggman.tv/c/s-ruby-advanced), 这个课程录的时间有点早，不过 Ruby 从 2.0 到现在的 3.x 变化不大主要是性能的提升，课程里面讲述的内容有点多，催眠向，但是坚持看完肯定收获会不少，早期录的，如有让您不舒服的地方还请见谅！[eggmantv/ruby_advanced: Ruby高级开发 http://eggman.tv/c/s-ruby-advanced (github.com)](https://github.com/eggmantv/ruby_advanced)
2. Ruby 日常，[https://eggman.tv/c/s-learning-ruby-everyday](https://eggman.tv/c/s-learning-ruby-everyday), 内容不长，看的会比较快，之前本来计划这个专题是周更的，后来搁置了。
3. Ruby 元编程，[https://eggman.tv/c/s-ruby-meta-programming](https://eggman.tv/c/s-ruby-meta-programming), 高级主题，不建议实际开发中经常用，但是要了解
4. Ruby 元编程实战, [https://eggman.tv/c/s-ruby-metaprogramming-in-action](https://eggman.tv/c/s-ruby-metaprogramming-in-action), 不多说了，学习了上面的，这个不学其实也行
5. Ruby 大师之路，[https://eggman.tv/c/s-ruby-road-to-master](https://eggman.tv/c/s-ruby-road-to-master), 课程名字唬人的，实际开发用处不多，但是学了装逼还是可以的

**Ruby on Rails 学习路线**

1. Ruby on Rails 高级开发，[https://eggman.tv/c/s-rails-advanced](https://eggman.tv/c/s-rails-advanced), 同上面的**Ruby 高级开发**，都是早期录的，催眠但干货多（搭配 https://guides.rubyonrails.org/getting_started.html 一起看）[eggmantv/rails_advanced](https://github.com/eggmantv/rails_advanced)
2. Rails 实战之 B2C 商城开发, [https://eggman.tv/c/s-master-rails-by-actions](https://eggman.tv/c/s-master-rails-by-actions), 实战为主，建议结合上面的一起看
3. Rails 5 源码分析, [https://eggman.tv/c/s-digging-into-rails-5-source-code](https://eggman.tv/c/s-digging-into-rails-5-source-code), 虽然 Rails 已经出到 7 了，但是这个还是可以一看的，源码分析的更多目的是为了学习代码的设计理念和扩展思路，并不是让你也这样写代码，关键是你也这样写代码，别的小伙伴可能就看不懂了
4. 其他的根据兴趣随便看吧

## Ruby Introduction（介绍）

- Ruby 是开源的面向对象程序设计的编程语言
- 在 20 世纪 90 年代中期由日本的松本行弘（Matz）开发
- Ruby 可运行于多种平台，如 Windows、macOS、Linux 的各种版本
- Ruby 流行起来的根本原因是基于 Ruby 的 Web 开发框架 Rails 的广泛使用
- 国内真正开始使用 Ruby 是在 2006 年左右

### 特点

- 解释型（不需要编译可运行）
- 泛型（不需要显式声明，更动态和灵活）
- OOP（更彻底的面向对象编程语言）
- Module（模块）
- 反射
- 垃圾回收（有自身的 GC 机制）
- 为**程序员**而生，而不是计算机
	- 性能问题
	- 团队合作

Everything is an Object、Lightweight、Cross platform

## 代码演示

**Everything is an Object**

```rb
# 10 也是个对象
10.times do |t|
  puts "#{t} hello world"
end
```

上述代码依次打印出 数字 (0-9) hello world

可以在 [run.rb - Run Ruby Online (runrb.io)](https://runrb.io/) 网站运行代码

```rb
puts "hello world".is_a?(String) # true
```

方法链调用

```rb
puts "hello world".gsub(/world/, 'Ruby').upcase # HELLO RUBY
```

反向查找，查找方法并过滤

```rb
puts "hello world".methods.grep(/case/)

# 以下为输出结果
casecmp
casecmp?
downcase
upcase
swapcase
upcase!
downcase!
swapcase!
```

**语法**

```rb
a = 1
b = 2.34

name = "Tom"

firstName = :Frank # Symbol 符号

persons = ["rose", "jack", "andy"] # 数组

info = {"name" => "Tom", "age" => 20} # hash 字典
info = {name: "Tom", age: 20}

reg = /hello/i # 正则

range = 1..10 # 1到10

# 代码块
calculator = proc { |x, y| x + y }

# 方法
def say_hi_to(name)
end

# 类
class Bird
end

# 模块
module BirdHelpers
end

# 判断
if var1
# do something
elsif var2
# do something
else
# do something
end

a = 1 unless defined? a # unless = if not

b = 2 if a == 1

case var3
when 1
# do something
when 2
# do something
when /hello/
# do something
else
# do something
end

# 迭代
for x in [1, 2, 3, 4] do
  puts x
end

[1, 2, 4].each do |x|
  puts x
end

x = 10
while x > 0 do
  puts x
  x -= 1
end

# 异常捕获
begin
  # do something
  puts "执行代码"
rescue
  # exception handling
  puts "捕获异常"
else
  # if there's no exception...
  puts "没有异常"
ensure
  # whether or not exception happens, do this...
  puts "一定执行"
end

# 元编程，尤其在 rspec 的时候
Site.should_receive('get_original_servers').with("http://www.google.com").and_return(original_servers)
response.body.should == original_servers.to_json

```

## Ruby on Rails

基于 Ruby 的 web 框架

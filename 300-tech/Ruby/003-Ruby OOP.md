---
title: 003-Ruby OOP
date: 2024-07-16 10:59
updated: 2024-07-16 10:59
---

## 面向对象基础

设计模式：怎样写代码（写代码的思路和方式）

例如：人有属性（attribute）和行为（方法 method）

- 年龄、性别、身高等等
- 爱好、睡觉等等

特殊人群会有特殊的属性和特殊行为，可以继承自人后扩展

还有函数式编程等

## Ruby 中的面向对象

Everything is Object

```rb
a = "hello" # 等同于  a = String.new("hello")
p a.class # String

b = 3.14
p b.class # Float

c = %w[pear cat horse]
p c.class # Array
```

`String#length`

```rb
a = "hello"
a.length # 5
```

length 被称为 String 的一个实例方法（instance method）

## 实例方法和实例变量

- Instance Method：实例方法、成员方法、成员函数
- Instance Attribute：实例变量、成员属性、属性（property），使用 `@` 定义

```rb
class User # 大写字母开头
  def initialize name, age # 内部特殊的方法，有点类似其他语言里的构造函数
    @name = name # 声明实例变量
    @age = age # @开头的正常变量
  end
  
  # getter 实例方法
  def name
    @name # 返回内部的实例变量
  end
  def age
    @age
  end
  # setter
  def name= name
    @name = name
  end
  def age= age
    @age = age
  end
end

user = User.new('Tom', 18) # 实例化
p user.name # 调用实例方法
p user.age

user.name = "Ruby"
user.age = 10
p user.name
p user.age
```

Ruby 提供了自动生成的

```rb
class User
  attr_reader :name
  attr_reader :age
  # 可以写在一行  attr_reader :name, :age
  
  attr_writer :name
  attr_writer :age
  # 可以写在一行  attr_writer :name, :age

  # 上面的可以合写成
  # attr_accessor :name, :age
  
  def initialize name, age
    @name = name
    @age = age
  end

  def say_hi
    p "I am  #{@name}, #{age} year old."
  end
end

user = User.new('Tom', 18)
p user.name
user.say_hi
```

`#{age}` 引用的是读的方法

## 类方法和类变量

- Class Method：类方法、静态方法
- Class Variable：类变量，使用 `@@` 定义

不会随实例而变化，类似其他语言的 `static`

```rb
class User
  attr_accessor :name, :age
  
  @@counter = 0 # 类变量
  
  def initialize name, age
    @name = name
    @age = age
    
    @@counter += 1
  end
  
  def say_hi
    p "I am  #{@name}, #{age} year old."
  end
  
  # 类方法
  def self.get_counter # def User.get_counter      User 类自身的方法，不是实例方法
    @@counter
    # 访问不到实例方法、实例变量
  end
end

user = User.new 'Tom', 18
p user.name # "Tom"
user1 = User.new 'Jock', 12
p User.get_counter # 2
```

## 访问控制

Access Control 一般针对实例方法

- public
- protected 保护的，外部不能访问，子类能继承
- private 私有的，外部和子类继承都不能访问

```rb
class User
  attr_accessor :name, :age
  
  def initialize name, age
    @name = name
    @age = age
  end
  
  public # 实例外部能够调用的
  def say_hello
    p "Hello"
    say_hi
    say_hey
  end
  
  protected
  def say_hi
    p "I am  #{@name}, #{age} year old."
  end
  
  private
  def say_hey
    p "Hey, I'm #{name}"
  end
end

user = User.new 'Tom', 18
user.say_hello
```

还可以不在定义的上面写，而是统一写在一起（类的 end 的上面）

```rb
protected :say_hi
private :say_hey
```


## Ruby 的动态特性

Dynamic Ruby

> `attr_accessor` 方法从哪里来的？

```rb
class User
end

User.class # Class
```

来自于 `Class` 类

`define_method` 能够动态定义方法

```rb
define_method :hello do
  puts "hello world"
end
hello
```

实现一下读写

```rb
class User
  #attr_accessor :name, :age
  # 不使用 Ruby 自带的，自己实现一下
  
  def self.setup_accessor var
    define_method var do
      instance_variable_get "@#{var}"
    end
    define_method "#{var}=" do |value|
      instance_variable_set "@#{var}", value
    end
  end
  
  setup_accessor :name
  setup_accessor :age
  
  def initialize name, age
    @name = name
    @age = age
  end
end

user = User.new('Tom', 18)
p user.name
p user.age

user.name = "Ruby"
user.age = 10
p user.name
p user.age
```

## 继承

Inheritance

例如：User → Admin、人 → 超人

```rb
class User
  attr_accessor :name, :age
  
  def initialize name, age
    @name, @age = name, age
  end
  
  def panels
    @panels ||= ['Profile', 'Products']
  end
end

class Admin < User
  def panels
    @panels ||= ['Profile', 'Products', 'Manage Users', 'System Setup']
  end
end

user = User.new('user_1', 18)
p user.panels # ["Profile", "Products"]

admin = Admin.new('admin_1', 20)
puts admin.name # admin_1
p admin.panels # ["Profile", "Products", "Manage Users", "System Setup"]
```

继承的子类可以声明新的属性和方法，也可以重新定义父类的属性和方法，这样将会覆盖父类的属性和方法

```rb
p Admin.superclass # 查看父类
```

super 关键字，可以**调用父类的同名方法**

```rb
class Admin < User
  def panels
    # @panels ||= ['Profile', 'Products', 'Manage Users', 'System Setup']
    super # 会调用父类的 panels 方法
    @panels.concat(['Manage Users', 'System Setup']) # 直接在后面追加
  end
end
```

super 可以传递参数

self 关键字，当前作用域指向的实例

```rb
class User
  attr_accessor :name, :age
  
  def initialize name, age
    @name, @age = name, age
  end
  
  def panels
    @panels ||= ['Profile', 'Products']
  end

  def to_s
    "#{self.name} #{self.age}" # 实例方法的 name age
    # "#{name} #{age}" # 当前作用域没有，向上层即当前实例查找就能找到 accessor 提供的方法
  end

  def self.category # 当前类
    'User'
  end

  class << self
    def category # 和上面等同 self.category
      'User'
    end
    # 定义在里面的都是 self.xxxx
  end
end

user = User.new('user_1', 18)
puts user
```

在 Ruby 中 class 没有多重继承的概念（只能继承一个类）

## 模块 Modules

作用

- namespace：避免命名冲突
- mixin：实现类似于多类继承的功能
- storage：不会经常变动的放到一个模块进行存储

关键字

- module：声明一个模块
- include：把一个模块的方法注入为实例方法
- extend：把一个模块的方法注入为类方法

```rb
class User
  attr_accessor :name, :age
  
  def initialize name, age
    @name, @age = name, age
  end
  
  def panels
    @panels ||= ['Profile', 'Products']
  end
end

# 声明一个模块
module Management
  def company_notifies
    'company_notifies'
  end
end

# module Module1
#...
# end

class Admin < User
  include Management # include 进来成实例方法
  
  def panels
    super
    @panels << ['Manage Users', 'System Setup']
  end
end

class Staff < User
  include Management
  # extend Module1 # 注入进来为类方法
  
  def panels
    super
    @panels << ['Tasks']
  end
end

admin = Admin.new('admin', 1)
puts admin.company_notifies
# Staff.Module1
```

## Class 进阶

Ruby 的内部类结构

```rb
Array.class # Class
Class.class # Class
```

superclass 父类/超类

```rb
Array.superclass # Object
Object.superclass # BasicObject
BasicObject.superclass # nil
```

ancestors 继承链 里面会有模块

```rb
Array.ancestors # [Array, Enumerable, Object, Kernel, BasicObject]
```

Method Finding 方法查找是自下而上的

```rb
# 先找自身有没有定义方法，没有就根据继承链往上查找
```

Method Overwrite 方法覆盖

- class 和 module 可以重新定义（如果有会覆盖，没有的就打补丁）
- 方法可以重新定义

```rb
class User
  def panels
    @panels ||= ['Profile', 'Products']
  end
end

class User
  def panels
    'overwrite'
  end
end

user = User.new
puts user.panels # overwrite
```

可以打开系统的类，进行方法增加，最好不要去覆盖内置的方法

```rb
class Array
  def to_hello_world # 所有数组上都有这个方法
      "hello #{self.join(', ')}"
  end
end

a = %w[cat dog mouse]
puts a.to_hello_world # hello cat, dog, mouse
```

## Module 进阶

Module 本质上也是一个 class

```rb
Array.ancestors # [Array, Enumerable, Object, Kernel, BasicObject]
Enumerable.class # Module
Module.class # Class
```

去除了实例化功能

```rb
module Management
  def company_notifies
    'company_notifies'
  end
end

class User
  include Management
  
  def company_notifies
    puts super # 调用父类的同名方法
    'company_notifies from user'
  end
end

p User.ancestors # [User, Management, Object, Kernel, BasicObject]
user = User.new
puts user.company_notifies
# company_notifies
# company_notifies from user
```

多个模块注入同样的方法，它将会根据 ancestors 最近的那个调用

模块可以 include 另一个模块

模块中定义类方法也是使用 self 关键字，使用 extend 注入

- `include` 把模块注入当前类的继承链**后面**
- `prepend` 把模块注入当前类的继承链**前面**

included 方法：当模块被 include 时会被执行，同时会传递当前作用域的 self 对象

```rb
module Management
  def self.included base
    puts "#{base}"
    
    base.include InstanceMethods
    base.extend ClassMethods
  end
  
  module InstanceMethods
    def company_notifies
      'company_notifies from management'
    end
  end
  
  module ClassMethods
    def progress
      'progress'
    end
  end
end

class User
  include Management
end

p User.ancestors
user = User.new
puts user.company_notifies
puts User.progress

# User
# [User, Management::InstanceMethods, Management, Object, Kernel, BasicObject]
# company_notifies from management
# progress
```


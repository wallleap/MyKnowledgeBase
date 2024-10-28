---
title: 021-Active Support
date: 2024-08-09T04:53:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

内置的 Gem 包 activesupport，它扩展了 Ruby 的内置类，并且提供了更多底层的功能

```sh
$ irb
3.0.0 :001 > "".empty?
 => true 
3.0.0 :002 > " ".empty?
 => false 
3.0.0 :003 > 
3.0.0 :004 > require 'active_support/all'
 => true 
3.0.0 :005 > " ".empty?
 => false 
3.0.0 :006 > " ".blank?
 => true
```

Rails 默认加载了 activesupport gem

```sh
$ bundle list | grep active
  * activejob (7.0.8.4)
  * activemodel (7.0.8.4)
  * activerecord (7.0.8.4)
  * activestorage (7.0.8.4)
  * activesupport (7.0.8.4)
```

## active_support 的引用

active_support 按照功能划分的模块，可以针对需要使用的方法或者模块来独立引用

```rb
require 'active_support'
require 'active_support/core_ext/object/blank'

" ".blank?
```

也可以引用所有的扩展方法和模块

```rb
require 'active_support/all'
```

Rails 默认加载了所有 active_support 的扩展

## 基础扩展

- `blank?`
- `present?`
- `deep_dup`
- `try`
- `to_query`
- `to_json`

```sh
$ rails c
Loading development environment (Rails 7.0.8.4)
3.0.0 :001 > "".blank?
 => true 
3.0.0 :002 > [].blank?
 => true 
3.0.0 :003 > {}.blank?
 => true 
3.0.0 :004 > nil.blank?
 => true 
3.0.0 :005 > 
3.0.0 :006 > nil.present?
 => false 
3.0.0 :007 > " ".present?
 => false 
3.0.0 :008 > {b: 1}.present?
 => true 
3.0.0 :009 > 
3.0.0 :010 > a = ["a", "b"]
 => ["a", "b"] 
3.0.0 :011 > b = a
 => ["a", "b"] 
3.0.0 :012 > b.first.sub!("a", "A")
 => "A" 
3.0.0 :013 > b
 => ["A", "b"] 
3.0.0 :014 > a
 => ["A", "b"] 
3.0.0 :015 > b = a.dup
 => ["A", "b"] 
3.0.0 :016 > b.second.sub!("b", "B")
 => "B" 
3.0.0 :017 > b
 => ["A", "B"] 
3.0.0 :018 > a
 => ["A", "B"] 
3.0.0 :019 > b = a.deep_dup
 => ["A", "B"] 
3.0.0 :020 > b.first.sub!("A", "a")
 => "a" 
3.0.0 :021 > b
 => ["a", "B"] 
3.0.0 :023 > a
 => ["A", "B"] 
3.0.0 :024 > 
3.0.0 :025 > nil.size
(irb):25:in `<main>': undefined method `size' for nil:NilClass (NoMethodError)
3.0.0 :026 > nil.try(:size)
 => nil 
3.0.0 :027 > 
3.0.0 :030 > {a:1, b:2}.to_query
 => "a=1&b=2"
3.0.0 :031 > {a:1, b:2}.to_json
 => "{\"a\":1,\"b\":2}" 
3.0.0 :032 > 
3.0.0 :033 > class A
3.0.0 :034 >   def initialize
3.0.0 :035 >     @name = "luwang"
3.0.0 :036 >     @title = "ruby"
3.0.0 :037 >   end
3.0.0 :038 > end
 => :initialize 
3.0.0 :039 > a = A.new
 => #<A:0x000000011f3539d0 @name="luwang", @title="ruby"> 
3.0.0 :040 > a.instance_values
 => {"name"=>"luwang", "title"=>"ruby"} 
3.0.0 :041 > a.instance_variable_names
 => ["@name", "@title"] 
3.0.0 :042 >
3.0.0 :044 > "hello".in? ["hello"]
 => true 
3.0.0 :045 > 3.in? ["hello", 3]
 => true 
3.0.0 :046 > [1, 2].include? 1
 => true 
3.0.0 :047 > 
3.0.0 :049 > "hello world".truncate 5
 => "he..."
3.0.0 :050 > "hello world".starts_with? "h"
 => true 
3.0.0 :050 > "hello world".starts_with? "h"
 => true 
3.0.0 :053 > "apple".pluralize
 => "apples" 
3.0.0 :054 > "apple".pluralize(2)
 => "apples" 
3.0.0 :055 > "apple".pluralize(1)
 => "apple"
3.0.0 :056 > "apples".singularize
 => "apple"
3.0.0 :058 > "hello_world".camelize
 => "HelloWorld" 
3.0.0 :060 > "blog".camelize.constantize
 => Blog (call 'Blog.connection' to establish a connection) 
3.0.0 :061 > "blog".camelize.constantize.new
 => #<Blog:0x000000013fd1c8c0 id: nil, title: nil, content: nil, is_public: false, user_id: nil, created_at: nil, updated_at: nil> 
3.0.0 :062 > "blog_comment".camelize.constantize
(irb):62:in `<main>': uninitialized constant BlogComment (NameError)
Did you mean?  BlogsController
3.0.0 :063 > 
3.0.0 :064 > a = { :a=>1, :b=>2 }
 => {:a=>1, :b=>2} 
3.0.0 :065 > a.stringify_keys
 => {"a"=>1, "b"=>2} 
3.0.0 :066 > a.stringify_keys.symbolize_keys
 => {:a=>1, :b=>2}
3.0.0 :069 > Time.now
 => 2024-08-09 17:39:13.881122 +0800 
3.0.0 :070 > "2024-08-09".to_date
 => Fri, 09 Aug 2024 
3.0.0 :071 > "2024-08-09".to_date + 1.year
 => Sat, 09 Aug 2025 
3.0.0 :072 > "2024-08-09".to_date + 1.month
 => Mon, 09 Sep 2024 
3.0.0 :073 > 3.days.ago
 => Tue, 06 Aug 2024 09:40:19.728159000 UTC +00:00 
3.0.0 :074 > Time.now.beginning_of_week
 => 2024-08-05 00:00:00 +0800 
3.0.0 :075 > Time.now.beginning_of_month
 => 2024-08-01 00:00:00 +0800 
3.0.0 :076 > Time.now.end_of_month
 => 2024-08-31 23:59:59.999999999 +0800 
```

## Class 扩展

- `class_attribute`
- `cattr_read` / `cattr_writer` / `cattr_accessor`
- `subclasses`

```sh
3.0.0 :078 > module B
3.0.0 :079 >   mattr_accessor :name
3.0.0 :080 > end
 => :name= 
3.0.0 :081 > B.name
 => nil 
3.0.0 :082 > B.name = 'luwang'
 => "luwang" 
3.0.0 :083 > B.name
 => "luwang" 
3.0.0 :084 > 
3.0.0 :085 > class C
3.0.0 :086 >   class_attribute :name
3.0.0 :087 >   self.name = "nickname"
3.0.0 :088 > end
 => "nickname" 
3.0.0 :089 > C.name
 => "nickname" 
```

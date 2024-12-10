---
title: Ruby 异常处理最佳实践
date: 2024-11-14T10:17:55+08:00
updated: 2024-11-14T10:19:20+08:00
dg-publish: false
---

『异常处理』是个比较容易忽视的话题，特别是 ruby/rails 圈，很多写了很久代码的同学恐怕都没有怎么写过异常处理的代码。实际上，异常处理还是挺重要的，想写出『健壮』的代码，必须得了解清楚异常的机制以及异常处理的最佳实践。

关于异常处理，问题无非是下面几个，并且各个高级语言的异常处理大同小异，使用上没有太大区别：

1. 异常是为了解决什么问题而出现的？
2. 异常都有哪些类型？分别应该怎么应对？
3. 什么时候需要使用异常？什么时候需要捕获异常？
4. 使用异常的最佳实践有哪些？

这篇文章并不会说明 ruby 异常的语法啊使用啊什么的，只是尝试回答上面的问题，期望让大家对异常有一个更深入的认识，不过我自己对异常的认识也没有那么深刻，所以有不对的地方，欢迎指出。

另外，文后的参考文章中都讲的很不错，推荐阅读。

## 为什么要使用异常（Exception）？

有些语言本身不提供异常机制，比如 C 语言，那么方法/函数之间的通信一般是用错误码（error code）来实现（可以自行 google 一下 error code vs exception），后来慢慢的高级语言中的 Exception 才成为标配。一般新的特性的出现总是为了解决某些问题的，Exception 解决了什么问题，或者说它带来了哪些好处？

1. Exception 可以附带更多的信息

	 错误码一般是全局预定义好的一批，然后进行一些归类，系统中所有的函数统一使用这些错误码，函数出现异常的时候使用错误码来告知调用方有异常出现。每一个错误码对应一个唯一的信息。

	 而异常这种方式本质上是一个类/对象，那么异常本身就可以被继承、被封装，也可以添加自己的方法，自己的属性，所以它的信息量就会更大、更灵活；

2. Exception 处理是跳跃式的

	 这个应该是最重要的特性，支持异常处理的语言中，异常被抛出之后，程序的运行逻辑直接跳到相应的 catch 这个异常的地方继续执行。这中间可能经过了多次的函数调用，但是并不影响异常的执行。

	 这条规则的好处在于，不用所有的函数都要考虑去捕获异常，只要在某个清楚知道如何处理改异常的地方去处理就好了。

3. Exception 的机制很清晰的独立了异常处理的代码和正常的业务逻辑

	 因为 throw/catch 的机制，所以可以很清晰的独立开异常处理的逻辑和正常的业务逻辑，这个比较容易理解。

## 什么时候应该使用异常？

新手一般很难搞清楚异常到底是怎么定义的，什么情况下应该算是『异常』？是不是每遇到一个什么错误都应该抛一个异常？

首先，异常一般指下面三种情况：

1. 程序错误

	 也就是程序写的有问题，比如 java 中最常见的 NullPointerException, ruby 中的 ZeroDivisionError、NoMethodError 等等。

	 这类错误一般不用去捕获，一旦发生时让程序挂起即可。通过一些测试和 debug，找到相应的问题修正代码逻辑即可。比如 NullPointerException，那么通过计算或者处理下标，把下标控制在 size 的范围内即可。

2. 调用方式不正确导致的错误

	 每一个函数/类/URL，某种程度上，都可以理解为是一种 API，是调用方和实现方的接口。如果调用方传入了不符合约定的参数，就会导致错误发生。比如典型情况，参数需要一段 XML，但是传入的参数并不是一个有效的 xml，就会导致一个异常发生。

	 这种情况下，最好的处理方式是告知调用方哪里出错了，这样 client 就可以去重试。可以通过异常机制处理，也可以通过简单的 if/else 去处理。

3. 运行时资源出现异常

	 这种是最多的情况，一般需要处理的异常大多是这种情况。比如网络调用时网络不通了、申请内存时系统没有内存了，都属于『运行时资源异常』的情况。

	 这时一般是通过抛异常的形式，由底层抛出异常，在业务层重新封装并提供合适的信息，最终由上层去处理。

## 使用异常的最佳实践

使用异常的最佳实践不太好总结，其实只有是两个问题最重要：

1. 我要不要抛一个异常？
2. 要不要捕获某个异常？

所以，在编码的时候，每当需要 raise 或者 catch 一个异常，就可以想想这两个问题，然后用下面的原则来判断，相信会得到更好的答案。

### 什么时候抛一个异常？

1. 参数校验可以提前做，避免不必要的异常

	 比如在用户注册时，可能手机号已经被注册过了，昵称被占用了，这种情况很有可能发生，一般情况下直接使用参数校验，然后返回给客户端即可，无需使用异常。

	 这种情况，属于上面的参数错误，这种情况一般通过参数的校验，提前返回效果更好。

2. 有时候应该属于正常的业务逻辑，不应该使用异常

	 比如电商系统中，预订某件商品的时候一定会遇到库存不足的情况，这种情况应该先判断库存是否足够，库存不足的时候直接返回即可。这种情况一般是不需要使用异常来进行处理的。

	 这种处理方式也更接近于 ruby 的哲学，比如 java 中数组越界会提示 NullPointerException，但是 ruby 中直接返回 nil，表示没有这个值。

3. 编程错误，一般需要 raise 异常

	 比如常见的父类的虚函数，一般也会在父类中定义一个接口：

	 ```
   class Parent
       def foo
           raise NotImplementedError, "need implementation in child class"
       end
   end
   
   class Child
       def foo
           "bar"
       end
   end
   ```

	 这种时候，如果写程序的时候调用了父类的 foo 函数，直接抛出异常。

4. 框架内的某些特殊约定，使用异常才能完成某项功能

	 比如：

\* rails 事务中，需要抛出异常，事务才会回滚； * sneaker 中，需要抛出异常，消息才不会被消费，然后可以重试；

1. 发生资源异常时，一般业务层需要『转译』异常

	 日常工作中，我们很少需要写很底层的库，而资源类的异常一般是比较底层的库抛出的，比如网络异常、文件系统异常、内存异常、锁异常等。对于日常写的更多的业务层的代码，一般会把这类异常进行转译（把原来的异常封装一下，更换为一个新的类）。这样的好处在于可以提供更多的信息，也可以更好的对类做分层（上层的代码只会依赖于紧邻的下一层）。

	 一个简单的例子来自于 rails/active_record 的源码，我们知道 ar 封装了很多读取数据库的 API，比如下面的代码在数据库连接不可用的时候会抛出异常：

	 ```
   class Order < ActiveRecord::Base
   end
   
   # 此时，数据库挂掉的话，调用 find 会发生什么？
   o = Order.find(12345)
   ```

	 异常打出的提示是这样的：

	 > ActiveRecord::StatementInvalid: Mysql2::Error: Lost connection to MySQL server during query: SELECT `orders`.* FROM `orders` WHERE `orders`.`id` = 5 LIMIT 1

	 可以很明显的看到，Mysql2 gem 抛了一个异常，ActiveRecord 对它进行了『转译』。

	 不过 ruby 不支持异常的直接封装，所以转译的时候只能把 message 给封装进来。下面是 rails 的实现：

	 ```
   # active_record/connection_adapters/abstract_adapter.rb，省略了部分代码
   def log(sql, name = "SQL", binds = [])
     @instrumenter.instrument(....) { yield }
   rescue Exception => e
     message = "#{e.class.name}: #{e.message}: #{sql}"
     exception = translate_exception(e, message) # 转译异常
     exception.set_backtrace e.backtrace # 存储 backtrace
     raise exception
   end
   
   def translate_exception(e, message)
     # override in derived class
     ActiveRecord::StatementInvalid.new(message)
   end
   ```

不过什么时候抛出异常，争议也比较多，最基本的前提是在一个系统内保持一致的行为。如果约定某些情况下抛出异常，那么整个系统都应该抛出异常。

### 什么时候需要捕获异常？

这个原则上就比较简单了，只有在『你有能力处理的时候才去捕获异常』，否则就不要去捕获。

如果再细节一点的话，可能下面的一些原则可以帮助写出更健壮的程序：

1. 不要直接吞掉异常

	 就是 rescue 之后什么也不错，事实上，这种情况还经常出现。如果真觉得应该在这一层捕获异常，那么至少打点日志吧。（有了日志，运维同学才可以监控、可以报警啊喂）

	 ```
   begin
       do(something)
   rescue => e
       Rails.logger.error("something is wrong: #{e.message}")
       Rails.logger.debug e.backtrace.join('\n')
   end
   ```

2. 尽量捕获更细节的异常，不要直接捕获 Exception 这种顶级异常

	 也就是上面的原则，能处理什么异常就捕获什么异常。异常机制的最终目的是进行某种程度的 recovery，所以只对自己能处理的异常进行捕获即可。

3. 异常也是普通的类，所以也应纳入到分层体系中去

	 也就是上面提到的『转译』异常，比如在业务层，可能需要对异常进行封装，然后由上层来捕获。而不是直接由上层的代码直接捕获底层的异常。

	 一般来说，资源类的错误是在实现层去触发，在中间层进行转译，然后在上层进行捕获和处理。

异常的使用本身就有一些争议，但是日常工作中肯定避免不了使用异常，所以还是要对异常有一定的理解的，异常也能帮助大家写出更健壮的程序~

以上。

## 参考文章

1. <http://www.onjava.com/pub/a/onjava/2003/11/19/exceptions.html>
2. <http://howtodoinjava.com/best-practices/java-exception-handling-best-practices/>
3. <https://msdn.microsoft.com/en-us/library/hh279678.aspx>
4. <http://www.monkeyandcrow.com/blog/reading_rails_handling_exceptions/>
5. <http://code.tutsplus.com/articles/writing-robust-web-applications-the-lost-art-of-exception-handling--net-36395>

---

抛异常部分不好说，但是捕获异常觉得还是 `不应该把异常抛出给最终用户`，话说在 ApplicationController 里同意捕获 log 应该是比较常见的做法吧。

——rails 里面是这样的，一般用 rescue_from 来统一捕获部分异常然后进行处理，不过这个不是这篇文字讲的重点。 异常当然不应该抛给最终用户，抛异常的时候关心抛出合适的异常即可，至于是谁来捕获，就需要来看具体的业务逻辑了，一般是上层捕获下层的异常，直接处理掉或者进行转译再抛出。

---

应该参考一下 python 的处理方式，Easier to ask for forgiveness than permission，先斩后奏。 <https://docs.python.org/2/glossary.html#term-eafp>

这跟语言异常机制的实现也有关吧，python 的比较轻量。

——其实 ruby 的也算轻量了，不然可以看看 java 世界，还要区分 checked exception 和 unchecked exception，直接晕掉。 异常最重要的是合理使用，不能过度，也肯定避免不了不用

---

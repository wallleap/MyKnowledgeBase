---
title: Promise async Generator 实现原理解析
date: 2023-08-04 15:02
updated: 2023-08-04 15:07
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - Promise async Generator 实现原理解析
rating: 1
tags:
  - JS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

笔者刚接触 `async/await` 时，就被其暂停执行的特性吸引了，心想在没有原生 API 支持的情况下，await 居然能挂起当前方法，实现暂停执行，我感到十分好奇。好奇心驱使我一层一层剥开有关 JS 异步编程的一切。阅读完本文，读者应该能够了解：

1. `Promise` 的实现原理
2. `async/await` 的实现原理
3. `Generator` 的实现原理

## Promise 实现

在成文过程中，笔者查阅了很多讲解 Promise 实现的文章，但感觉大多文章都很难称得上条理清晰，有的上来就放大段 Promise 规范翻译，有的在 Promise 基础使用上浪费篇幅，又或者把一个简单的东西长篇大论，过度讲解，我推荐头铁的同学直接拉到本章小结看最终实现，结合着注释直接啃代码也能理解十之八九

回归正题，文章开头我们先点一下 Promise 为我们解决了什么问题：在传统的异步编程中，如果异步之间存在依赖关系，我们就需要通过层层嵌套回调来满足这种依赖，如果嵌套层数过多，可读性和可维护性都变得很差，产生所谓“回调地狱”，而 Promise 将回调嵌套改为链式调用，增加可读性和可维护性。下面我们就来一步步实现一个 Promise：

## 1\. 观察者模式

我们先来看一个最简单的 Promise 使用：

```js
const p1 = new Promise((resolve, reject) => {
    setTimeout(() => {
        resolve('result')
    },
    1000);
}) 

p1.then(res => console.log(res), err => console.log(err))
```

观察这个例子，我们分析 Promise 的调用流程：

- `Promise` 的构造方法接收一个 `executor()`，在 `new Promise()` 时就立刻执行这个 executor 回调
- `executor()` 内部的异步任务被放入宏/微任务队列，等待执行
- `then()` 被执行，收集成功/失败回调，放入成功/失败队列
- `executor()` 的异步任务被执行，触发 `resolve/reject`，从成功/失败队列中取出回调依次执行

其实熟悉设计模式的同学，很容易就能意识到这是个**观察者模式**，这种 `收集依赖 -> 触发通知 -> 取出依赖执行` 的方式，被广泛运用于观察者模式的实现，在 Promise 里，执行顺序是 `then收集依赖 -> 异步触发resolve -> resolve执行依赖`。依此，我们可以勾勒出 Promise 的大致形状：

```js
class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
    this._resolveQueue = []    // then收集的执行成功的回调队列
    this._rejectQueue = []     // then收集的执行失败的回调队列

    // 由于resolve/reject是在executor内部被调用, 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
      // 从成功队列里取出回调依次执行
      while(this._resolveQueue.length) {
        const callback = this._resolveQueue.shift()
        callback(val)
      }
    }
    // 实现同resolve
    let _reject = (val) => {
      while(this._rejectQueue.length) {
        const callback = this._rejectQueue.shift()
        callback(val)
      }
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调，并push进对应队列
  then(resolveFn, rejectFn) {
    this._resolveQueue.push(resolveFn)
    this._rejectQueue.push(rejectFn)
  }
}
```

写完代码我们可以测试一下:

```js
const p1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('result')
  }, 1000);
})
p1.then(res => console.log(res))
//一秒后输出result
```

我们运用观察者模式简单的实现了一下 `then` 和 `resolve`，使我们能够在 then 方法的回调里取得异步操作的返回值，但我们这个 Promise 离最终实现还有很长的距离，下面我们来一步步补充这个 Promise：

## 2\. Promise A+ 规范

上面我们已经简单地实现了一个超低配版 Promise，但我们会看到很多文章和我们写的不一样，他们的 Promise 实现中还引入了各种状态控制，这是由于 ES6 的 Promise 实现需要遵循 [Promise/A+规范](https://link.juejin.cn/?target=https%3A%2F%2Fpromisesaplus.com%2F "https://promisesaplus.com/")，是规范对 Promise 的状态控制做了要求。Promise/A+ 的规范比较长，这里只总结两条核心规则：

> 1.  Promise 本质是一个状态机，且状态只能为以下三种：`Pending（等待态）`、`Fulfilled（执行态）`、`Rejected（拒绝态）`，状态的变更是单向的，只能从 Pending -> Fulfilled 或 Pending -> Rejected，状态变更不可逆
> 2.  `then方法` 接收两个可选参数，分别对应状态改变时触发的回调。then 方法返回一个 promise。then 方法可以被同一个 promise 调用多次。

![](https://cdn.wallleap.cn/img/pic/illustration/202308041503163.jpeg)

根据规范，我们补充一下 Promise 的代码：

```js
//Promise/A+规范的三种状态
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
    this._status = PENDING     // Promise状态
    this._resolveQueue = []    // 成功队列, resolve时触发
    this._rejectQueue = []     // 失败队列, reject时触发

    // 由于resolve/reject是在executor内部被调用, 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
      if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
      this._status = FULFILLED              // 变更状态

      // 这里之所以使用一个队列来储存回调,是为了实现规范要求的 "then 方法可以被同一个 promise 调用多次"
      // 如果使用一个变量而非队列来储存回调,那么即使多次p1.then()也只会执行一次回调
      while(this._resolveQueue.length) {    
        const callback = this._resolveQueue.shift()
        callback(val)
      }
    }
    // 实现同resolve
    let _reject = (val) => {
      if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
      this._status = REJECTED               // 变更状态
      while(this._rejectQueue.length) {
        const callback = this._rejectQueue.shift()
        callback(val)
      }
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    this._resolveQueue.push(resolveFn)
    this._rejectQueue.push(rejectFn)
  }
}
```

## 3\. then 的链式调用

补充完规范，我们接着来实现链式调用，这是 Promise 实现的重点和难点，我们先来看一下 then 是如何链式调用的：

```js
const p1 = new Promise((resolve, reject) => {
  resolve(1)
})

p1
  .then(res => {
    console.log(res)
    //then回调中可以return一个Promise
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve(2)
      }, 1000);
    })
  })
  .then(res => {
    console.log(res)
    //then回调中也可以return一个值
    return 3
  })
  .then(res => {
    console.log(res)
  })
```

输出

```js
1
2
3
```

我们思考一下如何实现这种链式调用：

1. 显然 `.then()` 需要返回一个 Promise，这样才能找到 then 方法，所以我们会把 then 方法的返回值包装成 Promise。
2. `.then()` 的回调需要拿到上一个 `.then()` 的返回值
3. `.then()` 的回调需要顺序执行，以上面这段代码为例，虽然中间 return 了一个 Promise，但执行顺序仍要保证是 1->2->3。我们要等待当前 Promise 状态变更后，再执行下一个 then 收集的回调，这就要求我们对 then 的返回值分类讨论

```js
// then方法
then(resolveFn, rejectFn) {
  //return一个新的promise
  return new MyPromise((resolve, reject) => {
    //把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
    const fulfilledFn = value => {
      try {
        //执行第一个(当前的)Promise的成功回调,并获取返回值
        let x = resolveFn(value)
        //分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
        //这里resolve之后，就能被下一个.then()的回调获取到返回值，从而实现链式调用
        x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
      } catch (error) {
        reject(error)
      }
    }
    //把后续then收集的依赖都push进当前Promise的成功回调队列中(_rejectQueue), 这是为了保证顺序调用
    this._resolveQueue.push(fulfilledFn)

    //reject同理
    const rejectedFn  = error => {
      try {
        let x = rejectFn(error)
        x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
      } catch (error) {
        reject(error)
      }
    }
    this._rejectQueue.push(rejectedFn)
  })
}
```

然后我们就能测试一下链式调用：

```js
const p1 = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve(1)
  }, 500);
})

p1
  .then(res => {
    console.log(res)
    return 2
  })
  .then(res => {
    console.log(res)
    return 3
  })
  .then(res => {
    console.log(res)
  })

//输出 1 2 3
```

## 4.值穿透 & 状态已变更的情况

我们已经初步完成了链式调用，但是对于 then() 方法，我们还要两个细节需要处理一下

1. **值穿透**：根据规范，如果 then() 接收的参数不是 function，那么我们应该忽略它。如果没有忽略，当 then() 回调不为 function 时将会抛出异常，导致链式调用中断
2. **处理状态为 resolve/reject 的情况**：其实我们上边 then() 的写法是对应状态为 `padding` 的情况，但是有些时候，resolve/reject 在 then() 之前就被执行（比如 `Promise.resolve().then()`），如果这个时候还把 then() 回调 push 进 resolve/reject 的执行队列里，那么回调将不会被执行，因此对于状态已经变为 `fulfilled` 或 `rejected` 的情况，我们直接执行 then 回调：

```js
// then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    // 根据规范，如果then的参数不是function，则我们需要忽略它, 让链式调用继续往下执行
    typeof resolveFn !== 'function' ? resolveFn = value => value : null
    typeof rejectFn !== 'function' ? rejectFn = reason => {
      throw new Error(reason instanceof Error? reason.message:reason);
    } : null
  
    // return一个新的promise
    return new MyPromise((resolve, reject) => {
      // 把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
      const fulfilledFn = value => {
        try {
          // 执行第一个(当前的)Promise的成功回调,并获取返回值
          let x = resolveFn(value)
          // 分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      // reject同理
      const rejectedFn  = error => {
        try {
          let x = rejectFn(error)
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      switch (this._status) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        case PENDING:
          this._resolveQueue.push(fulfilledFn)
          this._rejectQueue.push(rejectedFn)
          break;
        // 当状态已经变为resolve/reject时,直接执行then回调
        case FULFILLED:
          fulfilledFn(this._value)    // this._value是上一个then回调return的值(见完整版代码)
          break;
        case REJECTED:
          rejectedFn(this._value)
          break;
      }
    })
  }
```

## 5.兼容同步任务

完成了 then 的链式调用以后，我们再处理一个前边的细节，然后放出完整代码。上文我们说过，Promise 的执行顺序是 `new Promise -> then()收集回调 -> resolve/reject执行回调`，这一顺序是建立在**executor 是异步任务**的前提上的，如果 executor 是一个同步任务，那么顺序就会变成 `new Promise -> resolve/reject执行回调 -> then()收集回调`，resolve 的执行跑到 then 之前去了，为了兼容这种情况，我们给 `resolve/reject` 执行回调的操作包一个 setTimeout，让它异步执行。

> 这里插一句，有关这个 setTimeout，其实还有一番学问。虽然规范没有要求回调应该被放进宏任务队列还是微任务队列，但其实 Promise 的默认实现是放进了微任务队列，我们的实现（包括大多数 Promise 手动实现和 polyfill 的转化）都是使用 setTimeout 放入了宏任务队列（当然我们也可以用 MutationObserver 模拟微任务）

```js
//Promise/A+规定的三种状态
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
    this._status = PENDING     // Promise状态
    this._value = undefined    // 储存then回调return的值
    this._resolveQueue = []    // 成功队列, resolve时触发
    this._rejectQueue = []     // 失败队列, reject时触发

    // 由于resolve/reject是在executor内部被调用, 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
      //把resolve执行回调的操作封装成一个函数,放进setTimeout里,以兼容executor是同步代码的情况
      const run = () => {
        if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
        this._status = FULFILLED              // 变更状态
        this._value = val                     // 储存当前value

        // 这里之所以使用一个队列来储存回调,是为了实现规范要求的 "then 方法可以被同一个 promise 调用多次"
        // 如果使用一个变量而非队列来储存回调,那么即使多次p1.then()也只会执行一次回调
        while(this._resolveQueue.length) {    
          const callback = this._resolveQueue.shift()
          callback(val)
        }
      }
      setTimeout(run)
    }
    // 实现同resolve
    let _reject = (val) => {
      const run = () => {
        if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
        this._status = REJECTED               // 变更状态
        this._value = val                     // 储存当前value
        while(this._rejectQueue.length) {
          const callback = this._rejectQueue.shift()
          callback(val)
        }
      }
      setTimeout(run)
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    // 根据规范，如果then的参数不是function，则我们需要忽略它, 让链式调用继续往下执行
    typeof resolveFn !== 'function' ? resolveFn = value => value : null
    typeof rejectFn !== 'function' ? rejectFn = reason => {
      throw new Error(reason instanceof Error? reason.message:reason);
    } : null
  
    // return一个新的promise
    return new MyPromise((resolve, reject) => {
      // 把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
      const fulfilledFn = value => {
        try {
          // 执行第一个(当前的)Promise的成功回调,并获取返回值
          let x = resolveFn(value)
          // 分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      // reject同理
      const rejectedFn  = error => {
        try {
          let x = rejectFn(error)
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      switch (this._status) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        case PENDING:
          this._resolveQueue.push(fulfilledFn)
          this._rejectQueue.push(rejectedFn)
          break;
        // 当状态已经变为resolve/reject时,直接执行then回调
        case FULFILLED:
          fulfilledFn(this._value)    // this._value是上一个then回调return的值(见完整版代码)
          break;
        case REJECTED:
          rejectedFn(this._value)
          break;
      }
    })
  }
}
```

然后我们可以测试一下这个 Promise：

```js
const p1 = new MyPromise((resolve, reject) => {
  resolve(1)          //同步executor测试
})

p1
  .then(res => {
    console.log(res)
    return 2          //链式调用测试
  })
  .then()             //值穿透测试
  .then(res => {
    console.log(res)
    return new MyPromise((resolve, reject) => {
      resolve(3)      //返回Promise测试
    })
  })
  .then(res => {
    console.log(res)
    throw new Error('reject测试')   //reject测试
  })
  .then(() => {}, err => {
    console.log(err)
  })

// 输出 
// 1 
// 2 
// 3 
// Error: reject测试
```

到这里，我们已经实现了 Promise 的主要功能 `(｀∀´)Ψ` 剩下的几个方法都非常简单，我们顺手收拾掉：

## Promise.prototype.catch()

> `catch()方法` 返回一个 Promise，并且处理拒绝的情况。它的行为与调用 Promise.prototype.then(undefined, onRejected) 相同。

```js
//catch方法其实就是执行一下then的第二个回调
catch(rejectFn) {
  return this.then(undefined, rejectFn)
}
```

## Promise.prototype.finally()

> `finally()方法` 返回一个 Promise。在 promise 结束时，无论结果是 fulfilled 或者是 rejected，都会执行指定的回调函数。在 finally 之后，还可以继续 then。并且会将值原封不动的传递给后面的 then

```js
//finally方法
finally(callback) {
  return this.then(
    value => MyPromise.resolve(callback()).then(() => value),             // MyPromise.resolve执行回调,并在then中return结果传递给后面的Promise
    reason => MyPromise.resolve(callback()).then(() => { throw reason })  // reject同理
  )
}
```

PS. 有同学问我 `MyPromise.resolve(callback())` 的意义，这里补充解释一下：这个写法其实涉及到一个 `finally()` 的使用细节，**finally() 如果 return 了一个 reject 状态的 Promise，将会改变当前 Promise 的状态**，这个 `MyPromise.resolve` 就用于改变 Promise 状态，在 finally() 没有返回 reject 态 Promise 或 throw 错误的情况下，去掉 `MyPromise.resolve` 也是一样的（欢迎大家向我提问，勘误的过程中也能很好地加深自己对 Promise 的理解，大家可以在各个交流群里直接@我）

> 参考资料：[对 Promise.prototype.finally() 的粗浅理解](https://link.juejin.cn/?target=https%3A%2F%2Fwww.jianshu.com%2Fp%2Ff0b94daf9bf7 "https://www.jianshu.com/p/f0b94daf9bf7")

## Promise.resolve()

> `Promise.resolve(value)` 方法返回一个以给定值解析后的 Promise 对象。如果该值为 promise，返回这个 promise；如果这个值是 thenable（即带有 "then" 方法)），返回的 promise 会“跟随”这个 thenable 的对象，采用它的最终状态；否则返回的 promise 将以此值完成。此函数将类 promise 对象的多层嵌套展平。

```js
//静态的resolve方法
static resolve(value) {
  if(value instanceof MyPromise) return value // 根据规范, 如果参数是Promise实例, 直接return这个实例
  return new MyPromise(resolve => resolve(value))
}
```

## Promise.reject()

> `Promise.reject()` 方法返回一个带有拒绝原因的 Promise 对象。

```js
//静态的reject方法
static reject(reason) {
  return new MyPromise((resolve, reject) => reject(reason))
}
```

## Promise.all()

> `Promise.all(iterable)` 方法返回一个 Promise 实例，此实例在 iterable 参数内所有的 promise 都“完成（resolved）”或参数中不包含 promise 时回调完成（resolve）；如果参数中 promise 有一个失败（rejected），此实例回调失败（reject），失败原因的是第一个失败 promise 的结果。

```js
//静态的all方法
static all(promiseArr) {
  let index = 0
  let result = []
  return new MyPromise((resolve, reject) => {
    promiseArr.forEach((p, i) => {
      //Promise.resolve(p)用于处理传入值不为Promise的情况
      MyPromise.resolve(p).then(
        val => {
          index++
          result[i] = val
          //所有then执行后, resolve结果
          if(index === promiseArr.length) {
            resolve(result)
          }
        },
        err => {
          //有一个Promise被reject时，MyPromise的状态变为reject
          reject(err)
        }
      )
    })
  })
}
```

## Promise.race()

> `Promise.race(iterable)` 方法返回一个 promise，一旦迭代器中的某个 promise 解决或拒绝，返回的 promise 就会解决或拒绝。

```js
static race(promiseArr) {
  return new MyPromise((resolve, reject) => {
    //同时执行Promise,如果有一个Promise的状态发生改变,就变更新MyPromise的状态
    for (let p of promiseArr) {
      MyPromise.resolve(p).then(  //Promise.resolve(p)用于处理传入值不为Promise的情况
        value => {
          resolve(value)        //注意这个resolve是上边new MyPromise的
        },
        err => {
          reject(err)
        }
      )
    }
  })
}
```

## 完整代码

```js
//Promise/A+规定的三种状态
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  // 构造方法接收一个回调
  constructor(executor) {
    this._status = PENDING     // Promise状态
    this._value = undefined    // 储存then回调return的值
    this._resolveQueue = []    // 成功队列, resolve时触发
    this._rejectQueue = []     // 失败队列, reject时触发

    // 由于resolve/reject是在executor内部被调用, 因此需要使用箭头函数固定this指向, 否则找不到this._resolveQueue
    let _resolve = (val) => {
      //把resolve执行回调的操作封装成一个函数,放进setTimeout里,以兼容executor是同步代码的情况
      const run = () => {
        if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
        this._status = FULFILLED              // 变更状态
        this._value = val                     // 储存当前value

        // 这里之所以使用一个队列来储存回调,是为了实现规范要求的 "then 方法可以被同一个 promise 调用多次"
        // 如果使用一个变量而非队列来储存回调,那么即使多次p1.then()也只会执行一次回调
        while(this._resolveQueue.length) {    
          const callback = this._resolveQueue.shift()
          callback(val)
        }
      }
      setTimeout(run)
    }
    // 实现同resolve
    let _reject = (val) => {
      const run = () => {
        if(this._status !== PENDING) return   // 对应规范中的"状态只能由pending到fulfilled或rejected"
        this._status = REJECTED               // 变更状态
        this._value = val                     // 储存当前value
        while(this._rejectQueue.length) {
          const callback = this._rejectQueue.shift()
          callback(val)
        }
      }
      setTimeout(run)
    }
    // new Promise()时立即执行executor,并传入resolve和reject
    executor(_resolve, _reject)
  }

  // then方法,接收一个成功的回调和一个失败的回调
  then(resolveFn, rejectFn) {
    // 根据规范，如果then的参数不是function，则我们需要忽略它, 让链式调用继续往下执行
    typeof resolveFn !== 'function' ? resolveFn = value => value : null
    typeof rejectFn !== 'function' ? rejectFn = reason => {
      throw new Error(reason instanceof Error? reason.message:reason);
    } : null
  
    // return一个新的promise
    return new MyPromise((resolve, reject) => {
      // 把resolveFn重新包装一下,再push进resolve执行队列,这是为了能够获取回调的返回值进行分类讨论
      const fulfilledFn = value => {
        try {
          // 执行第一个(当前的)Promise的成功回调,并获取返回值
          let x = resolveFn(value)
          // 分类讨论返回值,如果是Promise,那么等待Promise状态变更,否则直接resolve
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      // reject同理
      const rejectedFn  = error => {
        try {
          let x = rejectFn(error)
          x instanceof MyPromise ? x.then(resolve, reject) : resolve(x)
        } catch (error) {
          reject(error)
        }
      }
  
      switch (this._status) {
        // 当状态为pending时,把then回调push进resolve/reject执行队列,等待执行
        case PENDING:
          this._resolveQueue.push(fulfilledFn)
          this._rejectQueue.push(rejectedFn)
          break;
        // 当状态已经变为resolve/reject时,直接执行then回调
        case FULFILLED:
          fulfilledFn(this._value)    // this._value是上一个then回调return的值(见完整版代码)
          break;
        case REJECTED:
          rejectedFn(this._value)
          break;
      }
    })
  }

  //catch方法其实就是执行一下then的第二个回调
  catch(rejectFn) {
    return this.then(undefined, rejectFn)
  }

  //finally方法
  finally(callback) {
    return this.then(
      value => MyPromise.resolve(callback()).then(() => value),             //执行回调,并returnvalue传递给后面的then
      reason => MyPromise.resolve(callback()).then(() => { throw reason })  //reject同理
    )
  }

  //静态的resolve方法
  static resolve(value) {
    if(value instanceof MyPromise) return value //根据规范, 如果参数是Promise实例, 直接return这个实例
    return new MyPromise(resolve => resolve(value))
  }

  //静态的reject方法
  static reject(reason) {
    return new MyPromise((resolve, reject) => reject(reason))
  }

  //静态的all方法
  static all(promiseArr) {
    let index = 0
    let result = []
    return new MyPromise((resolve, reject) => {
      promiseArr.forEach((p, i) => {
        //Promise.resolve(p)用于处理传入值不为Promise的情况
        MyPromise.resolve(p).then(
          val => {
            index++
            result[i] = val
            if(index === promiseArr.length) {
              resolve(result)
            }
          },
          err => {
            reject(err)
          }
        )
      })
    })
  }

  //静态的race方法
  static race(promiseArr) {
    return new MyPromise((resolve, reject) => {
      //同时执行Promise,如果有一个Promise的状态发生改变,就变更新MyPromise的状态
      for (let p of promiseArr) {
        MyPromise.resolve(p).then(  //Promise.resolve(p)用于处理传入值不为Promise的情况
          value => {
            resolve(value)        //注意这个resolve是上边new MyPromise的
          },
          err => {
            reject(err)
          }
        )
      }
    })
  }
}
```

洋洋洒洒 150 多行的代码，到这里，我们终于可以给 Promise 的实现做一个结尾了。我们从一个最简单的 Promise 使用实例开始，通过对调用流程的分析，根据观察者模式实现了 Promise 的大致骨架，然后依据 Promise/A+ 规范填充代码，重点实现了 then 的链式调用，最后完成了 Promise 的静态/实例方法。其实 Promise 实现在整体上并没有太复杂的思想，但我们日常使用的时候往往忽略了很多 Promise 细节，因而很难写出一个符合规范的 Promise 实现，源码的实现过程，其实也是对 Promise 使用细节重新学习的过程。

## async/await 实现

虽然前边花了这么多篇幅讲 Promise 的实现，不过探索 `async/await` 暂停执行的机制才是我们的初衷，下面我们就来进入这一块的内容。同样地，开头我们点一下 async/await 的使用意义。 在多个回调依赖的场景中，尽管 Promise 通过链式调用取代了回调嵌套，但过多的链式调用可读性仍然不佳，流程控制也不方便，ES7 提出的 async 函数，终于让 JS 对于异步操作有了终极解决方案，简洁优美地解决了以上两个问题。

> 设想一个这样的场景，异步任务 a-> b-> c 之间存在依赖关系，如果我们通过 then 链式调用来处理这些关系，可读性并不是很好，如果我们想控制其中某个过程，比如在某些条件下，b 不往下执行到 c，那么也不是很方便控制

```js
Promise.resolve(a)
  .then(b => {
    // do something
  })
  .then(c => {
    // do something
  })
```

> 但是如果通过 async/await 来实现这个场景，可读性和流程控制都会方便不少。

```js
async () => {
  const a = await Promise.resolve(a);
  const b = await Promise.resolve(b);
  const c = await Promise.resolve(c);
}
```

那么我们要如何实现一个 async/await 呢，首先我们要知道，**async/await 实际上是对 Generator（生成器）的封装**，是一个语法糖。由于 Generator 出现不久就被 async/await 取代了，很多同学对 Generator 比较陌生，因此我们先来看看 Generator 的用法：

> ES6 新引入了 Generator 函数，可以通过 yield 关键字，把函数的执行流挂起，通过 next() 方法可以切换到下一个状态，为改变执行流程提供了可能，从而为异步编程提供解决方案。

```js
function* myGenerator() {
  yield '1'
  yield '2'
  return '3'
}

const gen = myGenerator();  // 获取迭代器
gen.next()  //{value: "1", done: false}
gen.next()  //{value: "2", done: false}
gen.next()  //{value: "3", done: true}
```

也可以通过给 `next()` 传参, 让 yield 具有返回值

```js
function* myGenerator() {
  console.log(yield '1')  //test1
  console.log(yield '2')  //test2
  console.log(yield '3')  //test3
}

// 获取迭代器
const gen = myGenerator();

gen.next()
gen.next('test1')
gen.next('test2')
gen.next('test3')
```

我们看到 Generator 的用法，应该️会感到很熟悉，`*/yield` 和 `async/await` 看起来其实已经很相似了，它们都提供了暂停执行的功能，但二者又有三点不同：

- `async/await` 自带执行器，不需要手动调用 next() 就能自动执行下一步
- `async` 函数返回值是 Promise 对象，而 Generator 返回的是生成器对象
- `await` 能够返回 Promise 的 resolve/reject 的值

**我们对 async/await 的实现，其实也就是对应以上三点封装 Generator**

## 1.自动执行

我们先来看一下，对于这样一个 Generator，手动执行是怎样一个流程

```js
function* myGenerator() {
  yield Promise.resolve(1);
  yield Promise.resolve(2);
  yield Promise.resolve(3);
}

// 手动执行迭代器
const gen = myGenerator()
gen.next().value.then(val => {
  console.log(val)
  gen.next().value.then(val => {
    console.log(val)
    gen.next().value.then(val => {
      console.log(val)
    })
  })
})

//输出1 2 3
```

我们也可以通过给 `gen.next()` 传值的方式，让 yield 能返回 resolve 的值

```js
function* myGenerator() {
  console.log(yield Promise.resolve(1))   //1
  console.log(yield Promise.resolve(2))   //2
  console.log(yield Promise.resolve(3))   //3
}

// 手动执行迭代器
const gen = myGenerator()
gen.next().value.then(val => {
  // console.log(val)
  gen.next(val).value.then(val => {
    // console.log(val)
    gen.next(val).value.then(val => {
      // console.log(val)
      gen.next(val)
    })
  })
})
```

显然，手动执行的写法看起来既笨拙又丑陋，我们希望生成器函数能自动往下执行，且 yield 能返回 resolve 的值，基于这两个需求，我们进行一个基本的封装，这里 `async/await` 是关键字，不能重写，我们用函数来模拟：

```js
function run(gen) {
  var g = gen()                     //由于每次gen()获取到的都是最新的迭代器,因此获取迭代器操作要放在_next()之前,否则会进入死循环

  function _next(val) {             //封装一个方法, 递归执行g.next()
    var res = g.next(val)           //获取迭代器对象，并返回resolve的值
    if(res.done) return res.value   //递归终止条件
    res.value.then(val => {         //Promise的then方法是实现自动迭代的前提
      _next(val)                    //等待Promise完成就自动执行下一个next，并传入resolve的值
    })
  }
  _next()  //第一次执行
}
```

对于我们之前的例子，我们就能这样执行：

```js
function* myGenerator() {
  console.log(yield Promise.resolve(1))   //1
  console.log(yield Promise.resolve(2))   //2
  console.log(yield Promise.resolve(3))   //3
}

run(myGenerator)
```

这样我们就初步实现了一个 `async/await`。上边的代码只有五六行，但并不是一下就能看明白的，我们之前用了四个例子来做铺垫，也是为了让读者更好地理解这段代码。 简单来说，我们封装了一个 run 方法，run 方法里我们把执行下一步的操作封装成 `_next()`，每次 Promise.then() 的时候都去执行 `_next()`，实现自动迭代的效果。在迭代的过程中，我们还把 resolve 的值传入 `gen.next()`，使得 yield 得以返回 Promise 的 resolve 的值

> 这里插一句，是不是只有 `.then方法` 这样的形式才能完成我们自动执行的功能呢？答案是否定的，yield 后边除了接 Promise，还可以接 `thunk函数`，thunk 函数不是一个新东西，所谓 thunk 函数，就是**单参的只接受回调的函数**，详细介绍可以看 [阮一峰Thunk 函数的含义和用法](https://link.juejin.cn/?target=https%3A%2F%2Fp1-jj.byteimg.com%2Ftos-cn-i-t2oaga2asx%2Fgold-user-assets%2F2020%2F3%2F15%2F170dc5e88df6c208~tplv-t2oaga2asx-image.image "https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/3/15/170dc5e88df6c208~tplv-t2oaga2asx-image.image")，无论是 Promise 还是 thunk 函数，其核心都是通过**传入回调**的方式来实现 Generator 的自动执行。thunk 函数只作为一个拓展知识，理解有困难的同学也可以跳过这里，并不影响后续理解。

## 2.返回 Promise & 异常处理

虽然我们实现了 Generator 的自动执行以及让 yield 返回 resolve 的值，但上边的代码还存在着几点问题：

1. **需要兼容基本类型**：这段代码能自动执行的前提是 `yield` 后面跟 Promise，为了兼容后面跟着基本类型值的情况，我们需要把 yield 跟的内容 (`gen().next.value`) 都用 `Promise.resolve()` 转化一遍
2. **缺少错误处理**：上边代码里的 Promise 如果执行失败，就会导致后续执行直接中断，我们需要通过调用 `Generator.prototype.throw()`，把错误抛出来，才能被外层的 try-catch 捕获到
3. **返回值是 Promise**：`async/await` 的返回值是一个 Promise，我们这里也需要保持一致，给返回值包一个 Promise

我们改造一下 run 方法：

```js
function run(gen) {
  //把返回值包装成promise
  return new Promise((resolve, reject) => {
    var g = gen()

    function _next(val) {
      //错误处理
      try {
        var res = g.next(val) 
      } catch(err) {
        return reject(err); 
      }
      if(res.done) {
        return resolve(res.value);
      }
      //res.value包装为promise，以兼容yield后面跟基本类型的情况
      Promise.resolve(res.value).then(
        val => {
          _next(val);
        }, 
        err => {
          //抛出错误
          g.throw(err)
        });
    }
    _next();
  });
}
```

然后我们可以测试一下：

```js
function* myGenerator() {
  try {
    console.log(yield Promise.resolve(1)) 
    console.log(yield 2)   //2
    console.log(yield Promise.reject('error'))
  } catch (error) {
    console.log(error)
  }
}

const result = run(myGenerator)     //result是一个Promise
//输出 1 2 error
```

到这里，一个 `async/await` 的实现基本完成了。最后我们可以看一下 babel 对 async/await 的转换结果，其实整体的思路是一样的，但是写法稍有不同：

```js
//相当于我们的run()
function _asyncToGenerator(fn) {
  // return一个function，和async保持一致。我们的run直接执行了Generator，其实是不太规范的
  return function() {
    var self = this
    var args = arguments
    return new Promise(function(resolve, reject) {
      var gen = fn.apply(self, args);

      //相当于我们的_next()
      function _next(value) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'next', value);
      }
      //处理异常
      function _throw(err) {
        asyncGeneratorStep(gen, resolve, reject, _next, _throw, 'throw', err);
      }
      _next(undefined);
    });
  };
}

function asyncGeneratorStep(gen, resolve, reject, _next, _throw, key, arg) {
  try {
    var info = gen[key](arg);
    var value = info.value;
  } catch (error) {
    reject(error);
    return;
  }
  if (info.done) {
    resolve(value);
  } else {
    Promise.resolve(value).then(_next, _throw);
  }
}
```

使用方式：

```js
const foo = _asyncToGenerator(function* () {
  try {
    console.log(yield Promise.resolve(1))   //1
    console.log(yield 2)                    //2
    return '3'
  } catch (error) {
    console.log(error)
  }
})

foo().then(res => {
  console.log(res)                          //3
})
```

有关 `async/await` 的实现，到这里就告一段落了。但是直到结尾，我们也不知道 await 到底是如何暂停执行的，有关 await 暂停执行的秘密，我们还要到 Generator 的实现中去寻找答案

## Generator 实现

我们从一个简单的 Generator 使用实例开始，一步步探究 Generator 的实现原理：

```js
function* foo() {
  yield 'result1'
  yield 'result2'
  yield 'result3'
}
  
const gen = foo()
console.log(gen.next().value)
console.log(gen.next().value)
console.log(gen.next().value)
```

我们可以在 [babel官网](https://link.juejin.cn/?target=https%3A%2F%2Fbabeljs.io%2Frepl%2F%23%3Fbrowsers%3D%26build%3D%26builtIns%3Dfalse%26spec%3Dfalse%26loose%3Dfalse%26code_lz%3DGYVwdgxgLglg9mAVAAmHOAKAlMg3gKGWQE8YBTAGwBNkByAJzIGcQKoBGWwk86uxlmwBMXIqUo0GzVlADMXAL7d8EBEyjIA5mTDIAvKnTYVauBTIA6CnE0ZtYC2DIAPKNgsA3AIYUQZLCZgTGaW1rb2ji5uWJ4-fgGqQSFWNnY6ka7u3r7-QA%26debug%3Dfalse%26forceAllTransforms%3Dfalse%26shippedProposals%3Dfalse%26circleciRepo%3D%26evaluate%3Dfalse%26fileSize%3Dfalse%26timeTravel%3Dfalse%26sourceType%3Dmodule%26lineWrap%3Dtrue%26presets%3Des2015%252Creact%252Cstage-2%26prettier%3Dfalse%26targets%3D%26version%3D7.5.5%26externalPlugins%3D "https://babeljs.io/repl/#?browsers=&build=&builtIns=false&spec=false&loose=false&code_lz=GYVwdgxgLglg9mAVAAmHOAKAlMg3gKGWQE8YBTAGwBNkByAJzIGcQKoBGWwk86uxlmwBMXIqUo0GzVlADMXAL7d8EBEyjIA5mTDIAvKnTYVauBTIA6CnE0ZtYC2DIAPKNgsA3AIYUQZLCZgTGaW1rb2ji5uWJ4-fgGqQSFWNnY6ka7u3r7-QA&debug=false&forceAllTransforms=false&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=es2015%2Creact%2Cstage-2&prettier=false&targets=&version=7.5.5&externalPlugins=") 上在线转化这段代码，看看 ES5 环境下是如何实现 Generator 的：

```js
"use strict";

var _marked =
/*#__PURE__*/
regeneratorRuntime.mark(foo);

function foo() {
  return regeneratorRuntime.wrap(function foo$(_context) {
    while (1) {
      switch (_context.prev = _context.next) {
        case 0:
          _context.next = 2;
          return 'result1';

        case 2:
          _context.next = 4;
          return 'result2';

        case 4:
          _context.next = 6;
          return 'result3';

        case 6:
        case "end":
          return _context.stop();
      }
    }
  }, _marked);
}

var gen = foo();
console.log(gen.next().value);
console.log(gen.next().value);
console.log(gen.next().value);
```

代码咋一看不长，但如果仔细观察会发现有两个不认识的东西 —— `regeneratorRuntime.mark` 和 `regeneratorRuntime.wrap`，这两者其实是 regenerator-runtime 模块里的两个方法，regenerator-runtime 模块来自 facebook 的 regenerator 模块，完整代码在 [runtime.js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Ffacebook%2Fregenerator%2Fblob%2Fmaster%2Fpackages%2Fregenerator-runtime%2Fruntime.js "https://github.com/facebook/regenerator/blob/master/packages/regenerator-runtime/runtime.js")，这个 runtime 有 700 多行...-\_-||，因此我们不能全讲，不太重要的部分我们就简单地过一下，重点讲解暂停执行相关部分代码

> 个人觉得啃源码的效果不是很好，建议读者拉到末尾先看结论和简略版实现，源码作为一个补充理解

## regeneratorRuntime.mark()

`regeneratorRuntime.mark(foo)` 这个方法在第一行被调用，我们先看一下 runtime 里 mark() 方法的定义

```js
//runtime.js里的定义稍有不同，多了一些判断，以下是编译后的代码
runtime.mark = function(genFun) {
  genFun.__proto__ = GeneratorFunctionPrototype;
  genFun.prototype = Object.create(Gp);
  return genFun;
};
```

这里边 `GeneratorFunctionPrototype` 和 `Gp` 我们都不认识，他们被定义在 runtime 里，不过没关系，我们只要知道 `mark()方法` 为生成器函数（foo）绑定了一系列原型就可以了，这里就简单地过了

## regeneratorRuntime.wrap()

从上面 babel 转化的代码我们能看到，执行 `foo()`，其实就是执行 `wrap()`，那么这个方法起到什么作用呢，他想包装一个什么东西呢，我们先来看看 wrap 方法的定义：

```js
//runtime.js里的定义稍有不同，多了一些判断，以下是编译后的代码
function wrap(innerFn, outerFn, self) {
  var generator = Object.create(outerFn.prototype);
  var context = new Context([]);
  generator._invoke = makeInvokeMethod(innerFn, self, context);

  return generator;
}
```

wrap 方法先是创建了一个 generator，并继承 `outerFn.prototype`；然后 new 了一个 `context对象`；`makeInvokeMethod方法` 接收 `innerFn(对应foo$)`、`context` 和 `this`，并把返回值挂到 `generator._invoke` 上；最后 return 了 generator。**其实 wrap() 相当于是给 generator 增加了一个\_invoke 方法**

这段代码肯定让人产生很多疑问，outerFn.prototype 是什么，Context 又是什么，makeInvokeMethod 又做了哪些操作。下面我们就来一一解答：

> `outerFn.prototype` 其实就是 `genFun.prototype`，

这个我们结合一下上面的代码就能知道

> `context` 可以直接理解为这样一个全局对象，用于储存各种状态和上下文：

```js
var ContinueSentinel = {};

var context = {
  done: false,
  method: "next",
  next: 0,
  prev: 0,
  abrupt: function(type, arg) {
    var record = {};
    record.type = type;
    record.arg = arg;

    return this.complete(record);
  },
  complete: function(record, afterLoc) {
    if (record.type === "return") {
      this.rval = this.arg = record.arg;
      this.method = "return";
      this.next = "end";
    }

    return ContinueSentinel;
  },
  stop: function() {
    this.done = true;
    return this.rval;
  }
};
```

> `makeInvokeMethod` 的定义如下，它 return 了一个 `invoke方法`，invoke 用于判断当前状态和执行下一步，其实就是我们调用的 `next()`

```js
//以下是编译后的代码
function makeInvokeMethod(innerFn, context) {
  // 将状态置为start
  var state = "start";

  return function invoke(method, arg) {
    // 已完成
    if (state === "completed") {
      return { value: undefined, done: true };
    }
    
    context.method = method;
    context.arg = arg;

    // 执行中
    while (true) {
      state = "executing";

      var record = {
        type: "normal",
        arg: innerFn.call(self, context)    // 执行下一步,并获取状态(其实就是switch里边return的值)
      };

      if (record.type === "normal") {
        // 判断是否已经执行完成
        state = context.done ? "completed" : "yield";

        // ContinueSentinel其实是一个空对象,record.arg === {}则跳过return进入下一个循环
        // 什么时候record.arg会为空对象呢, 答案是没有后续yield语句或已经return的时候,也就是switch返回了空值的情况(跟着上面的switch走一下就知道了)
        if (record.arg === ContinueSentinel) {
          continue;
        }
        // next()的返回值
        return {
          value: record.arg,
          done: context.done
        };
      }
    }
  };
}
```

> 为什么 `generator._invoke` 实际上就是 `gen.next` 呢，因为在 runtime 对于 next() 的定义中，next() 其实就 return 了\_invoke 方法

```js
// Helper for defining the .next, .throw, and .return methods of the
// Iterator interface in terms of a single ._invoke method.
function defineIteratorMethods(prototype) {
    ["next", "throw", "return"].forEach(function(method) {
      prototype[method] = function(arg) {
        return this._invoke(method, arg);
      };
    });
}

defineIteratorMethods(Gp);
```

## 低配实现 & 调用流程分析

这么一遍源码下来，估计很多读者还是懵逼的，毕竟源码中纠集了很多概念和封装，一时半会不好完全理解，让我们跳出源码，实现一个简单的 Generator，然后再回过头看源码，会得到更清晰的认识

```js
// 生成器函数根据yield语句将代码分割为switch-case块，后续通过切换_context.prev和_context.next来分别执行各个case
function gen$(_context) {
  while (1) {
    switch (_context.prev = _context.next) {
      case 0:
        _context.next = 2;
        return 'result1';

      case 2:
        _context.next = 4;
        return 'result2';

      case 4:
        _context.next = 6;
        return 'result3';

      case 6:
      case "end":
        return _context.stop();
    }
  }
}

// 低配版context  
var context = {
  next:0,
  prev: 0,
  done: false,
  stop: function stop () {
    this.done = true
  }
}

// 低配版invoke
let gen = function() {
  return {
    next: function() {
      value = context.done ? undefined: gen$(context)
      done = context.done
      return {
        value,
        done
      }
    }
  }
} 

// 测试使用
var g = gen() 
g.next()  // {value: "result1", done: false}
g.next()  // {value: "result2", done: false}
g.next()  // {value: "result3", done: false}
g.next()  // {value: undefined, done: true}
```

这段代码并不难理解，我们分析一下调用流程：

1. 我们定义的 `function*` 生成器函数被转化为以上代码
2. 转化后的代码分为三大块：
		- `gen$(_context)` 由 yield 分割生成器函数代码而来
		- `context对象` 用于储存函数执行上下文
		- `invoke()方法` 定义 next()，用于执行 gen$(\_context) 来跳到下一步
3. 当我们调用 `g.next()`，就相当于调用 `invoke()方法`，执行 `gen$(_context)`，进入 switch 语句，switch 根据 context 的标识，执行对应的 case 块，return 对应结果
4. 当生成器函数运行到末尾（没有下一个 yield 或已经 return），switch 匹配不到对应代码块，就会 return 空值，这时 `g.next()` 返回 `{value: undefined, done: true}`

从中我们可以看出，**Generator 实现的核心在于 `上下文的保存`，函数并没有真的被挂起，每一次 yield，其实都执行了一遍传入的生成器函数，只是在这个过程中间用了一个 context 对象储存上下文，使得每次执行生成器函数的时候，都可以从上一个执行结果开始执行，看起来就像函数被挂起了一样**

## 总结 & 致谢

有关 Promise、async/await、Generator 的原理就实现到这里了，感谢大家能够跟我一起走完全程，不知不觉，我们花了近 9 千字来讲述有关异步编程的故事，异步编程的世界环环相扣，一开始，笔者只是出于对 await 挂起机制的好奇，后来，从一个 "await 是如何实现暂停执行 " 的小问题，引出了对异步编程的一系列思考和实现原理。三者的实现，其实也是前端异步编程一步步演化推进的过程。

成文过程中得到很多大佬的帮助，这四篇参考文章都是我阅读了很多相关文章后精选的四篇，建议大家结合阅读，大佬们写的比我好很多，另外感谢冴羽大佬在 Generator 机制上给予的解惑~

> 前端技匠：[各种源码实现，你想要的这里都有](https://juejin.cn/post/6844903989083897870 "https://juejin.cn/post/6844903989083897870")
> 神三元：[我如何实现Promise](https://juejin.cn/post/6844903702457925640 "https://juejin.cn/post/6844903702457925640")
> winty：[async/await 原理及执行顺序分析](https://juejin.cn/post/6844903988584775693#heading-0 "https://juejin.cn/post/6844903988584775693#heading-0")
> 冴羽：[ES6 系列之 Babel 将 Generator 编译成了什么样子](https://juejin.cn/post/6844903701908291592 "https://juejin.cn/post/6844903701908291592")

**最后卑微求个赞 Thanks♪(･ω･) ﾉ**

> **往期文章**
> 1.  [10行代码看尽redux实现 —— 全面剖析redux & react-redux & redux中间件设计实现 | 8k字](https://juejin.cn/post/6844904036013965325 "https://juejin.cn/post/6844904036013965325")
> 2.  [红黑树上红黑果，红黑树下你和我 —— 红黑树入门 | 6k字](https://juejin.cn/post/6844904006175686669 "https://juejin.cn/post/6844904006175686669")
> 3.  [SSR从入门到放弃 —— 深入React服务端渲染原理 | 1W字](https://juejin.cn/post/6844904000387563533 "https://juejin.cn/post/6844904000387563533")？

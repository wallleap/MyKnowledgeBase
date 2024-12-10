---
title: 手写 Promise
date: 2024-09-24T05:22:38+08:00
updated: 2024-10-21T03:49:27+08:00
dg-publish: false
---

手写 Promise 参考资料：

- Promise A+ 规范
- MDN 查看 JS 中其他方法

Promise A+ 规范，直接说了 promise 是一个对象或函数，拥有一个 `then` 方法

最核心的构造函数和实例上的 `then` 方法

## 构造函数

```js
// 状态常量
const PENDING = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  // 私有属性，外面不能用（_ 开头的是之前约定的私有变量）
  #state = PENDING
  #result = undefined
  #handlers = []

  constructor(executor) {  // 接收的是个函数 (resolve, reject) => {}
    // 两个属性，状态和结果，可以使用 ES6 的私有属性，这样外部就无法使用
    // this._state = 'pending'
    // this._result = undefined

    // 两个函数，new Promise 传入的函数 调用这两个函数会改变状态和结果
    // 如果写在外面（原型上），那么函数的 this 指向是 window，需要额外 bind
    const resolve = (data) => {
      this.#setState(FULFILLED, data)
    }
    const reject = (reason) => {
      this.#setState(REJECTED, reason)
    }
    // 手动 throw error 会导致 promise 状态变成 rejected
    try {
      executor(resolve, reject)  // 同步的，直接执行
    } catch (error) {  // 只能捕获同步错误，官方也捕获不了异步错误（setTimeout 中手动 throw）
      reject(error)
    }
  }

  #setState(state, result) {
    // 状态改变后不可逆
    if (this.#state !== PENDING) return
    this.#state = state
    this.#result = result
    // 还需要执行判断状态 then 的
    this.#run()
  }

  #isPromiseLike(value) {
    if (value !== null && typeof value === 'object' || typeof value === 'function') {
      return typeof value.then === 'function'
    } else {
      return false
    }
  }

  #runMicroTask(func) {
    if (typeof process === 'object' && typeof process.nextTick === 'function') {
      // node 环境，使用 process.nextTick 模拟微任务
      process.nextTick(func)
    } else if (typeof MutationObserver === 'function') {
      // 浏览器环境，使用观察器模拟微任务
      const ob = new MutationObserver(func)
      const textNode = document.createTextNode('')
      ob.observe(textNode, { characterData: true })
      textNode.data = '1'
    } else {
      setTimeout(func, 0)
    }
  }

  #runOne(callback, resolve, reject) {
    this.#runMicroTask(() => {
      if (typeof callback !== 'function') {
        const settled = this.#state === FULFILLED ? resolve : reject
        settled(this.#result)
        return
      }
      try {
        const data = callback(this.#result)
        // 3. 返回的是 promise
        if (this.#isPromiseLike(data)) {
          data.then(resolve, reject)
        } else {
          resolve(data)
        }
      } catch (error) {
        reject(error)
      }
    })
  }

  // then 中运行，状态改变后也会运行
  #run() {
    // 当前状态挂起，则结束
    if (this.#state === PENDING) return
    // 当前状态不是挂起，则从队列中取出回调函数，执行
    while (this.#handlers.length) {
      const { onFulfilled, onRejected, resolve, reject } = this.#handlers.shift()
      if (this.#state === FULFILLED) {
        /* if (typeof callback === 'function') {
          // 1. 执行回调是否报错
          try {
            const data = callback(this.#result)
            resolve(data)
          } catch (error) {
            reject(error)
          }
        } else {
          // 2. 对应的回调不是函数，穿透
          resolve(this.#result)
        } */
        // 下面一样抽离出去
        this.#runOne(onFulfilled, resolve, reject)
      } else {
        this.#runOne(onRejected, resolve, reject)
      }
    }
  }

  then(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {  // return 的是新的 promise，不是 this（this 的状态已经确定）
      // 可能是链式调用，但也可能是分开 p.then 然后再 p.then
      this.#handlers.push({
        onFulfilled,
        onRejected,
        resolve,
        reject
      })
      this.#run()
    })
  }
}
```

## 其他方法

ES6 还有 catch、finally、resolve、reject 等，查阅官方文档

```js
class MyPromise {
  ///...
  catch(onRejected) {
    return this.then(undefined, onRejected)
  }
  
  finally(onFinally) {
    return this.then(data => {
      onFinally()
      return data
    }, reason => {
      onFinally()
      throw reason
    })
  }

  static resolve(value) {
    if (value instanceof MyPromise) return value
    let _resolve, _reject
    const promise = new MyPromise((resolve, reject) => {
      _resolve = resolve
      _reject = reject
    })
    if (promise.#isPromiseLike(value)) {
      value.then(_resolve, _reject)
    } else {
      _resolve(value)
    }
    return promise
  }

  static reject(reason) {
    return new MyPromise((_, reject) => {
      reject(reason)
    })
  }

  static all(proms) {
    if (value instanceof MyPromise) return value
    let _resolve, _reject
    const promise = new MyPromise((resolve, reject) => {
      _resolve = resolve
      _reject = reject
    })
    const result = []
    let index = 0
    let fulfilled = 0
    for (const prom of proms) {
      const i = index
      index++
      MyPromise.resolve(prom).then((data) => {
        result[i] = data
        fulfilled++
        if (fulfilled === index) {
          _resolve(result)
        }
      }, _reject)
    }
    if (index === 0) {
      _resolve([])
    }
    return promise
  }
}
```

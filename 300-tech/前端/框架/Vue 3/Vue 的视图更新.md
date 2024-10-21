---
title: Vue 的视图更新
date: 2024-10-09T09:58:15+08:00
updated: 2024-10-09T09:58:22+08:00
dg-publish: false
---

是异步的，可以使用 `nextTick(() => {})` 在回调函数中获得更新后的结果

为什么需要是异步的

以下是同步的，在点击的时候写了两次 count++，effect 函数也触发了两次，没显示 1，直接显示 2，1 的显示是没必要的，会有性能损耗

```html
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <button id="btn">count++</button>
  <div id="el"></div>

  <script>
    const el = document.getElementById('el')
    const btn = document.getElementById('btn')

    // 正在执行的 effect
    let activeEffect = null
    // 保存依赖
    let set = new Set()

    function effect(fn) {
      // 保存当前的 effect
      activeEffect = fn
      fn()
      // 执行完之后清空
      activeEffect = null
    }

    const count = {
      _value: 0,
      get value() {
        // 收集依赖
        activeEffect && set.add(activeEffect)
        return this._value
      },
      set value(newValue) {
        this._value = newValue
        // 触发更新
        set.forEach(cb => cb())
      }
    }

    effect(() => {
      console.log('render')
      el.innerHTML = count.value
    })

    btn.onclick = () => {
      count.value++
      count.value++
    }
  </script>
</body>
</html>
```

不让它直接执行，而是使用 Set 存储任务，并且异步执行

```html
<script>
  const el = document.getElementById('el')
  const btn = document.getElementById('btn')

  // 正在执行的 effect
  let activeEffect = null
  // 保存依赖
  const set = new Set()
  // 存储待执行的任务，使用 Set 保证去重
  const tasks = new Set()

  function effect(fn) {
    // 保存当前的 effect
    activeEffect = fn
    fn()
    // 执行完之后清空
    activeEffect = null
  }

  function runTasks() {
    // 执行栈中的任务，异步
    Promise.resolve().then(() => {
      tasks.forEach(task => task())
      tasks.clear()
    })
  }

  function nextTick(cb) {
    Promise.resolve().then(cb)  // 放到微任务队列中，就会在 runTasks 后触发
  }

  const count = {
    _value: 0,
    get value() {
      // 收集依赖
      activeEffect && set.add(activeEffect)
      return this._value
    },
    set value(newValue) {
      this._value = newValue
      // 触发更新，先不直接执行
      set.forEach(cb => tasks.add(cb))
      runTasks()
    }
  }

  effect(() => {
    console.log('render')
    el.innerHTML = count.value
  })

  btn.onclick = () => {
    count.value++
    count.value++
    console.log('在 onclick 中', el.innerText)  // 拿到的还是 0，因为没有触发更新
    nextTick(() => {
      console.log('在 nextTick 中', el.innerText)  // 拿到的才是 2，因为触发了更新
    })
  }
</script>
```

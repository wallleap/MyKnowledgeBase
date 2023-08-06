---
title: 揭密 Vue 的双向绑定
date: 2023-08-04 17:55
updated: 2023-08-04 17:55
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 揭密 Vue 的双向绑定
rating: 1
tags:
  - Vue
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

Vue 中需要输入什么内容的时候，自然会想到使用 `<input v-model="xxx" />` 的方式来实现双向绑定。下面是一个最简单的示例

```
<div id="app">
    <h2>What's your name:</h2>
    <input v-model="name" />
    <div>Hello {{ name }}</div>
</div>

```

```
new Vue({
    el: "#app",
    data: {
       name: ""
    }
});
```

> **JsFiddle 演示**
> 
> [jsfiddle.net/0okxhc6f/](https://link.juejin.cn/?target=https%3A%2F%2Fjsfiddle.net%2F0okxhc6f%2F "https://jsfiddle.net/0okxhc6f/")

在这个示例的输入框中输入的内容，会随后呈现出来。这是 Vue 原生对 `<input>` 的良好支持，也是一个父组件和子组件之间进行双向数据传递的典型示例。不过 `v-model` 是 Vue 2.2.0 才加入的一个新功能，在此之前，Vue 只支持单向数据流。

## Vue 的单向数据流

Vue 的单向数据流和 React 相似，父组件可以通过设置子组件的属性（Props）来向子组件传递数据，而父组件想获得子组件的数据，得向子组件注册事件，在子组件高兴的时候触发这个事件把数据传递出来。一句话总结起来就是，Props 向下传递数据，事件向上传递数据。

上面那个例子，如果不使用 `v-model`，它应该是这样的

```
<input :value="name" @input="name = $event.target.value" />
```

由于事件处理写成了内联模式，所以脚本部分不需要修改。但是多数情况下，事件一般都会定义成一个方法，代码就会复杂得多

```
<input :value="name" @input="updateName" />
```

```
new Vue({
    // ....
    methods: {
        updateName(e) {
            this.name = e.target.value;
        }
    }
})
```

从上面的示例来看 `v-model` 节约了不少代码，最重要的是可以少定义一个事件处理函数。所以 `v-model` 实际干的事件包括

- 使用 `v-bind`（即 `:`）单向绑定一个属性（示例：`:value="name"`）
- 绑定 `input` 事件（即 `@input`）到一个默认实现的事件处理函数（示例：`@input=updateName`
- 这个默认的事件处理函数会根据事件对象带入的值来修改被绑定的数据（示例：`this.name = e.target.value`）

## 自定义组件的 `v-model`

Vue 对原生组件进行了封装，所以 `<input>` 在输入的时候会触发 `input` 事件。但是自定义组件应该怎么呢？这里不妨借助 JsFiddle Vue 样板的 Todo List 示例。

### JsFiddle 的 Vue 样板

> 点击 JsFilddle 的 Logo，在上面弹出面板中选择 Vue 样板即可

样板代码包含 HTML 和 Vue(js) 两个部分，代码如下：

```
<div id="app">
  <h2>Todos:</h2>
  <ol>
    <li v-for="todo in todos">
      <label>
        <input type="checkbox"
          v-on:change="toggle(todo)"
          v-bind:checked="todo.done">

        <del v-if="todo.done">
          {{ todo.text }}
        </del>
        <span v-else>
          {{ todo.text }}
        </span>
      </label>
    </li>
  </ol>
</div>

```

```
new Vue({
  el: "#app",
  data: {
    todos: [
      { text: "Learn JavaScript", done: false },
      { text: "Learn Vue", done: false },
      { text: "Play around in JSFiddle", done: true },
      { text: "Build something awesome", done: true }
    ]
  },
  methods: {
  toggle: function(todo){
    todo.done = !todo.done
    }
  }
})
```

### 定义 Todo 组件

JsFiddle 的 Vue 模板默认实现一个 Todo 列表的展示，数据是固定的，所有内容在一个模板中完成。我们首先要做事情是把单个 Todo 改成一个子组件。因为在 JsFiddle 中不能写成多文件的形式，所以组件使用 `Vue.component()` 在脚本中定义，主要是把 `<li>` 内容中的那部分拎出来：

```
Vue.component("todo", {
    template: `
<label>
    <input type="checkbox" @change="toggle" :checked="isDone">
    <del v-if="isDone">
        {{ text }}
    </del>
    <span v-else>
        {{ text }}
    </span>
</label>
`,
    props: ["text", "done"],
    data() {
        return {
            isDone: this.done
        };
    },
    methods: {
        toggle() {
            this.isDone = !this.isDone;
        }
    }
});
```

原来定义在 App 中的 `toggle()` 方法也稍作改动，定义在组件内了。`toggle()` 调用的时候会修改表示是否完成的 `done` 的值。但由于 `done` 是定义在 `props` 中的属性，不能直接赋值，所以采用了官方推荐的第一种方法，定义一个数据 `isDone`，初始化为 `this.done`，并在组件内使用 `isDone` 来控制是否完成这一状态。

相应的 App 部分的模板和代码精减了不少：

```
<div id="app">
    <h2>Todos:</h2>
    <ol>
        <li v-for="todo in todos">
            <todo :text="todo.text" :done="todo.done"></todo>
        </li>
    </ol>
</div>
```

```
new Vue({
    el: "#app",
    data: {
        todos: [
            { text: "Learn JavaScript", done: false },
            { text: "Learn Vue", done: false },
            { text: "Play around in JSFiddle", done: true },
            { text: "Build something awesome", done: true }
        ]
    }
});
```

> **JsFiddle 演示**
> 
> [jsfiddle.net/0okxhc6f/1/](https://link.juejin.cn/?target=https%3A%2F%2Fjsfiddle.net%2F0okxhc6f%2F1%2F "https://jsfiddle.net/0okxhc6f/1/")

不过到此为止，数据仍然是单向的。从效果上来看，点击复选框可以反馈出删除线线效果，但这些动态变化都是在 `todo` 组件内部完成的，不存在数据绑定的问题。

### 为 Todo List 添加计数

为了让 `todo` 组件内部的状态变化能在 Todo List 中呈现出来，我们在 Todo List 中添加计数，展示已经完成的 Todo 数量。因为这个数量受 `todo` 组件内部状态（数据）的影响，这就需要将 `todo` 内部数据变化反应到其父组件中，这才有 `v-model` 的用武之地。

这个数量我们在标题中以 `n/m` 的形式呈现，比如 `2/4` 表示一共 4 条 Todo，已经完成 2 条。这需要对 Todo List 的模板和代码部分进行修改，添加 `countDone` 和 `count` 两个计算属性：

```
<div id="app">
    <h2>Todos ({{ countDone }}/{{ count }}):</h2>
    <!-- ... -->
</div>

```

```
new Vue({
    // ...
    computed: {
        count() {
            return this.todos.length;
        },
        countDone() {
            return this.todos.filter(todo => todo.done).length;
        }
    }
});
```

现在计数呈现出来了，但是现在改变任务状态并不会对这个计数产生影响。我们要让子组件的变动对父组件的数据产生影响。`v-model` 待会儿再说，先用最常见的方法，事件：

- 子组件 `todo` 在 `toggle()` 中触发 `toggle` 事件并将 `isDone` 作为事件参数
- 父组件为子组件的 `toggle` 事件定义事件处理函数

```
Vue.component("todo", {
    //...
    methods: {
        toggle(e) {
            this.isDone = !this.isDone;
            this.$emit("toggle", this.isDone);
        }
    }
});
```

```
<!-- #app 中其它代码略 -->
<todo :text="todo.text" :done="todo.done" @toggle="todo.done = $event"></todo>
```

这里为 `@toggle` 绑定的是一个表达式。因为这里的 `todo` 是一个临时变量，如果在 `methods` 中定义专门的事件处理函数很难将这个临时变量绑定过去（当然定义普通方法通过调用的形式是可以实现的）。

> 事件处理函数，一般直接对应于要处理的事情，比如定义 `onToggle(e)`，绑定为 `@toggle="onToggle"`。这种情况下不能传入 `todo` 作为参数。
> 
> 普通方法，可以定义成 `toggle(todo, e)`，在事件定义中以函数调用表达式的形式调用：`@toggle="toggle(todo, $event)"。它和`todo.done = $event\` 同属表达式。
> 
> 注意二者的区别，前者是绑定的处理函数（引用），后者是绑定的表达式（调用）

现在通过事件方式已经达到了预期效果

> \*\* Js Fiddle 演示 \*\*
> 
> [jsfiddle.net/0okxhc6f/2/](https://link.juejin.cn/?target=https%3A%2F%2Fjsfiddle.net%2F0okxhc6f%2F2%2F "https://jsfiddle.net/0okxhc6f/2/")

### 改造成 `v-model`

之前我们说了要用 `v-model` 实现的，现在来改造一下。注意实现 `v-model` 的几个要素

- 子组件通过 `value` 属性（Prop）接受输入
- 子组件通过触发 `input` 事件输出，带数组参数
- 父组件中用 `v-model` 绑定

```
Vue.component("todo", {
    // ...
    props: ["text", "value"],   // <-- 注意 done 改成了 value
    data() {
        return {
            isDone: this.value    // <-- 注意 this.done 改成了 this.value
        };
    },
    methods: {
        toggle(e) {
            this.isDone = !this.isDone;
            this.$emit("input", this.isDone);  // <-- 注意事件名称变了
        }
    }
});
```

```
<!-- #app 中其它代码略 -->
<todo :text="todo.text" v-model="todo.done"></todo>
```

## `.sync` 实现其它数据绑定

前面讲到了 Vue 2.2.0 引入 `v-model` 特性。由于某些原因，它的输入属性是 `value`，但输出事件叫 `input`。`v-model`、`value`、`input` 这三个名称从字面上看不到半点关系。虽然这看起来有点奇葩，但这不是重点，重点是一个控件只能双向绑定一个属性吗？

Vue 2.3.0 引入了 `.sync` 修饰语用于修饰 `v-bind`（即 `:`），使之成为双向绑定。这同样是语法糖，添加了 `.sync` 修饰的数据绑定会像 `v-model` 一样自动注册事件处理函数来对被绑定的数据进行赋值。这种方式同样要求子组件触发特定的事件。不过这个事件的名称好歹和绑定属性名有点关系，是在绑定属性名前添加 `update:` 前缀。

比如 `<sub :some.sync="any" />` 将子组件的 `some` 属性与父组件的 `any` 数据绑定起来，子组件中需要通过 `$emit("update:some", value)` 来触发变更。

上面的示例中，使用 `v-model` 绑定始终感觉有点别扭，因为 `v-model` 的字面意义是双向绑定一个数值，而表示是否未完成的 `done` 其实是一个状态，而不是一个数值。所以我们再次对其进行修改，仍然使用 `done` 这个属性名称（而不是 `value`），通过 `.sync` 来实现双向绑定。

```
Vue.component("todo", {
    // ...
    props: ["text", "done"],   // <-- 恢复成 done
    data() {
        return {
            isDone: this.done    // <-- 恢复成 done
        };
    },
    methods: {
        toggle(e) {
            this.isDone = !this.isDone;
            this.$emit("update:done", this.isDone);  // <-- 事件名称：update:done
        }
    }
});
```

```
<!-- #app 中其它代码略 -->
<!-- 注意 v-model 变成了 :done.sync，别忘了冒号哟 -->
<todo :text="todo.text" :done.sync="todo.done"></todo>
```

> \*\* Js Fiddle 演示 \*\*
> 
> [jsfiddle.net/0okxhc6f/3/](https://link.juejin.cn/?target=https%3A%2F%2Fjsfiddle.net%2F0okxhc6f%2F3%2F "https://jsfiddle.net/0okxhc6f/3/")

## 揭密 Vue 双向绑定

通过上面的讲述，我想大家应该已经明白了 Vue 的双向绑定其实就是普通单向绑定和事件组合来完成的，只不过通过 `v-model` 和 `.sync` 注册了默认的处理函数来更新数据。Vue 源码中有这么一段

```
// @file: src/compiler/parser/index.js

if (modifiers.sync) {
    addHandler(
        el,
        `update:${camelize(name)}`,
        genAssignmentCode(value, `$event`)
    )
}
```

从这段代码可以看出来，`.sync` 双向绑定的时候，编译器会添加一个 `update:${camelize(name)}` 的事件处理函数来对数据进行赋值（`genAssignmentCode` 的字面意思是生成赋值的代码）。

## 展望

目前 Vue 的双向绑定还需要通过触发事件来实现数据回传。这和很多所的期望的赋值回传还是有一定的差距。造成这一差距的主要原因有两个

1. 需要通过事件回传数据
2. 属性（prop）不可赋值

在现在的 Vue 版本中，可以通过定义计算属性来实现简化，比如

```
computed: {
    isDone: {
        get() {
            return this.done;
        },
        set(value) {
            this.$emit("update:done", value);
        }
    }
}
```

说实在的，要多定义一个意义相同名称不同的变量名也是挺费脑筋的。希望 Vue 在将来的版本中可以通过一定的技术手段减化这一过程，比如为属性（Prop）声明添加 `sync` 选项，只要声明 `sync: true` 的都可以直接赋值并自动触发 `update:xxx` 事件。

当然作为一个框架，在解决一个问题的时候，还要考虑对其它特性的影响，以及框架的扩展性等问题，所以最终双向绑定会演进成什么样子，我们对 Vue 3.0 拭目以待。

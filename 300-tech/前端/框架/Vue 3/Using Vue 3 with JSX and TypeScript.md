---
title: Using Vue 3 with JSX and TypeScript
date: 2024-08-26T02:39:11+08:00
updated: 2024-08-26T02:39:39+08:00
dg-publish: false
source: https://www.bypaulshen.com/posts/vue-3-jsx-typescript
---

My friend and I are building a multiplayer drawing game with Vue. I used it as an excuse to learn and get familiar with Vue. We started with Vue 2 and templates but found that they don't scale well for complicated components.

I migrated the codebase to Vue 3 and now we're writing Vue components with JSX with TypeScript. There's relatively little documentation in the Vue 3 docs on JSX and TypeScript so I wanted to share some learnings that would've helped us. *Note that your app may not need JSX and TypeScript!*

This post augments the official Vue 3 documentation on [Render Functions](https://v3.vuejs.org/guide/render-function.html) and [TypeScript Support](https://v3.vuejs.org/guide/typescript-support.html). Check out those pages first if you haven't already.

Props

Types are inferred from the standard Vue component `props` definition. Vue comes with runtime prop checking with one of JavaScript's native constructors. Vue's TypeScript annotations will infer the prop's corresponding type.

- `String` -> `string`
- `Number` -> `number`
- `Boolean` -> `boolean`
- `Array` -> `unknown[]`
- `Object` -> `Record<string, any>`
- `Date` -> `string`
- `Function` -> `Function`
- `Symbol` -> `symbol`

You can cast the type using `PropType` to get a more specific TypeScript type. You will probably want to do this for every `Array`, `Object`, and `Function`.

```
enum Color {
  Red = 0,
  Green,
  Blue
}
const c = Color.Green; // this value is a JavaScript number
props: {
  color: {
    type: Number as PropType<Color>,
    required: true,
  },
  optionalNames: Array as PropType<string[]>,
}
```

**Props in Vue are optional by default.** Therefore, each prop will be `T | undefined` by default. If you specific a prop is required, the type will just be `T`.

```
props: {
  aNumber: Number,
  aRequiredNumber: {
    type: Number,
    required: true,
  }
},
setup(props: {
  aNumber?: number,
  aRequiredNumber: number,
}) {
}
```

This was not the case with Vue 2 and TypeScript. It confused me during the upgrade why I was getting so many new TypeScript prop errors. Eventually, I found [this code](https://github.com/vuejs/vue-next/blob/49bb44756fda0a7019c69f2fa6b880d9e41125aa/packages/runtime-core/src/componentProps.ts#L67-L90) that pointed to annotating props as required.

You cannot destructure props because the [the prop will lose reactivity](https://v3.vuejs.org/guide/composition-api-setup.html#props). You can use `toRefs` to convert the props to reactive values.

It may be tempting to `toRefs` all your props at the start of your `setup` so you have a consistent way to access props and reactive refs.

```
setup(props) {
  const {name, message} = toRefs(props);
  const counter = ref(0);
  // consistent access
  name.value;
  counter.value;
}
```

**Be careful when your prop is not required!** `toRefs` will not see the prop on `setup` and you will be working with a missing ref that is not reactive.

```
props: {
  count: Number, // this is not required
},
setup(props) {
  const { count } = toRefs(props);
  // The type of count is Ref<number | undefined> | undefined
  // This will not work! count (the ref) may be undefined.
  const countPlusOne = computed(() => (count?.value ?? 0) + 1);
}
```

For this reason, it may be better to always read props as `props.propName` without `toRefs`. Remember that proxies require a getter (`foo.bar`) to work. Without a getter, it's impossible for Vue to know when a variable has been accessed.

JSX

If you're building a component of significant or unknown complexity, I recommend you use [JSX](https://v3.vuejs.org/guide/render-function.html#jsx) (inside a `.tsx` file). I found Vue templates nice to start with but annoying as the file got larger. You lose a lot of expressiveness and your logic is spread throughout the file. For example, you can't use constants inside templates without exposing it in the component options or `setup`. Converting a Vue template -> JSX takes some effort.

I assumed the JSX transform was the same as React's (swapping `React.createElement` with `Vue.h`). This is close but Vue's JSX includes a few more changes.

It helped me to browse through Vue's [jsx-next repo](https://github.com/vuejs/jsx-next) and [JSX explorer](https://vue-next-jsx-explorer.netlify.app/). Of note, `v-` attributes are transformed differently. They are passed as directives to `Vue.h`. Some have custom transforms, such as `v-show`, `v-model`, and `v-slots`.

Here is an example of passing named slot children. View it in [the explorer](https://vue-next-jsx-explorer.netlify.app/#%3CMyComponent%0A%20%20count%3D%7B3%7D%0A%20%20v-message%3D%22hello%22%0A%20%20v-slots%3D%7B%7B%0A%20%20%20%20header%3A%20()%20%3D%3E%20%3Cdiv%3Eheader%3C%2Fdiv%3E%2C%0A%20%20%20%20body%3A%20()%20%3D%3E%20%3Cdiv%3Ebody%3C%2Fdiv%3E%2C%0A%20%20%7D%7D%0A%2F%3E).

```
<MyComponent
  v-slots={{
    header: () => <div>header</div>,
    body: () => <div>body</div>,
  }}
/>
```

This is the equivalent Vue template.

```
<MyComponent>
  <template v-slot:header>
    <div>header</div>
  </template>
  <template v-slot:body>
    <div>body</div>
  </template>
</MyComponent>
```

I don't believe there is a way to annotate types for directives like `v-show` and `v-slots`. Please correct me if I'm wrong.

Events

Events are more or less sugar on function props (i.e. a function prop named `onColorClick`) with validation. You can probably avoid Vue events completely and benefit from better type annotations.

Since we were converting Vue template components that emitted events, we wanted to know how to provide event callbacks in JSX. The way I figured this out was using the [Vue 3 Template Explorer](https://vue-next-template-explorer.netlify.app/). When you set an event handler like `@color-click="myCallback"` in Vue template, it's translated to a prop named `onColor-click`. More precisely, `@` is replaced with `on` and the first letter is capitalized. The rest is left alone (including dashes). You can find the relevant Vue compiler code [here](https://github.com/vuejs/vue-next/blob/a268f6fc7060956b1f312e50a5e5fd4436dcfd05/packages/compiler-core/src/parse.ts#L616-L618).

```
const MyComponent = defineComponent({
  setup(props, {emit}) {
    onMounted(() => {
      emit('dance');
      emit('party-time');
    });
  }
});
<MyComponent onDance={onDance} onParty-time={onPartyTime} />
```

I noticed that TypeScript will complain about `onDance` not being defined as a prop but `onParty-time` is okay. I believe this is because props with dashes are not validated as they are assumed to be DOM properties. Fix this by annotating the function prop in the component.

```
props: {
  onDance: Function as PropType<() => void>,
  "onParty-time": Function as PropType<() => void>,
}
```

Exposing a component API

With the component options API, you can expose methods that other components can call if they have a ref.

```
const MyComponent = {
  methods: {
    hello() { ... }
  }
};
```

If you're using the composition API and returning the render function in `setup`, how do you expose an instance method?

```
const MyComponent = {
  setup() {
    // how to expose a method?
    return () => <div />;
  }
};
```

The right answer is probably to move your render function out of your `setup` so you can expose the method on the return value.

```
const MyComponent = {
  setup() {
    return {
      hello() { ... }
    };
  },
  render() {
    return <div />;
  }
};
```

This seems like an unintentional side effect though. I like returning the render function in setup because you don't have to list out items to expose for your render function.

The following may not be recommended but it worked for us and seems like a reasonable alternative approach.

```
export type MyComponentAPI = {
  hello: (message: string) => void,
};
const MyComponent = {
  props: {
    instanceRef: Object as PropType<Ref<MyComponentAPI | undefined>>
  },
  setup(props) {
    if (props.instanceRef !== undefined) {
      // eslint-disable-next-line vue/no-mutating-props
      props.instanceRef.value = {
        hello(message: string) { ... }
      };
    }
    ...
  }
};
const OtherComponent = {
  setup() {
    const myComponentRef = ref<MyComponentAPI>();
    onMounted(() => {
      // with type checking!
      myComponentRef.value!.hello('bonjour');
    });
    return () => <MyComponent instanceRef={myComponentRef} />;
  }
};
```

There's sketchiness in mutating a prop ref but this approach gives you a typed instance API and lets you specify exactly what API the component is exposing.

Links

This post isn't comprehensive but covers some of the questions I had using Vue with TypeScript. When I have a question about Vue and TypeScript, my first destination is `node_modules/@vue/runtime-core/dist/runtime-core.d.ts`. The easiest way to get there is to go to definition (I'm using VSCode) on something Vue exports like `defineComponent`.

If I have a question about how to do something in JSX, I use the [Vue 3 Template Explorer](https://vue-next-template-explorer.netlify.app/) and [Vue 3 JSX Explorer](https://vue-next-jsx-explorer.netlify.app/). They compile to the same target so if you know how to do something with Vue templates, you can try to find the same JSX that compiles to similar JavaScript output. Good luck!

---
title: Vue 平滑移入指令
date: 2024-09-23T01:59:56+08:00
updated: 2024-09-23T05:28:08+08:00
dg-publish: false
---

做通用型的一定不要修改 style 属性

可以使用 Animation API

```js
const DISTANCE = 150
const DURATION = 1000

const animationMap = new WeakMap()

const ob = new IntersectionObserver(entries => {
  for (const entry of entries) {
    if (entry.isIntersecting) {
      const animation = animationMap.get(entry.target)
      animation.play()
      ob.unobserve(entry.target)
    }
  }
})

const isBelowViewport = el => (el.getBoundingClientRect().top > window.innerHeight)

export default {
  mounted(el) {
    if (!isBelowViewport(el)) return
    const animation = el.animate([
      {
        transform: `translateY(${DISTANCE}px)`,
        opacity: 0.5,
      },
      {
        transform: `translateY(0)`,
        opacity: 1,
      }
    ], {
      duration: DURATION,
      easing: 'ease',
    })
    animation.pause()
    animationMap.set(el, animation)
    ob.observe(el)
  },
  unmounted(el) {
    ob.unobserve(el)
  },
}
```

---
title: 07-封装 useSwipe Hook
date: 2024-05-16T03:55:00+08:00
updated: 2024-08-21T10:33:32+08:00
dg-publish: false
---

使用 Hooks，你能搞定一切需求

Vue 中叫 Composation API，但抄的 React 的 Hooks

新建 `hooks/useSwipe.tsx`

```tsx
import {computed, onMounted, onUnmounted, ref, Ref} from "vue"  
  
type Point = {  
  x: number  
  y: number  
}  
interface Options {  
  beforeStart?: (e: TouchEvent) => void  
  afterStart?: (e: TouchEvent) => void  
  beforeMove?: (e: TouchEvent) => void  
  afterMove?: (e: TouchEvent) => void  
  beforeEnd?: (e: TouchEvent) => void  
  afterEnd?: (e: TouchEvent) => void  
}  
  
export default (element: Ref<HTMLElement | null>, options: Options) => {  
  const start = ref<Point | null>(null)  
  const end = ref<Point | null>(null)  
  const swiping = ref(false)  
  const distance = computed(() => {  
    if (!start.value || !end.value) {  
      return null  
    }  
    return {  
      x: end.value.x - start.value.x,  
      y: end.value.y - start.value.y,  
    }  
  })  
  const direction = computed(() => {  
    if (!distance.value) {  
      return ''  
    }  
    const {x, y} = distance.value  
    if (Math.abs(x) > Math.abs(y)) {  
      return x > 0 ? 'right' : 'left'  
    } else {  
      return y > 0 ? 'down' : 'up'  
    }  
  })  
  const onStart = (e: TouchEvent) => {  
    options?.beforeStart?.(e)  
    swiping.value = true  
    end.value = start.value = {x: e.touches[0].screenX, y: e.touches[0].screenY}  
    options?.afterStart?.(e)  
  }  
  const onMove = (e: TouchEvent) => {  
    options?.beforeMove?.(e)  
    if (!start.value) {  
      return  
    }  
    end.value = {x: e.touches[0].screenX, y: e.touches[0].screenY}  
    options?.afterMove?.(e)  
  }  
  const onEnd = (e: TouchEvent) => {  
    options?.beforeEnd?.(e)  
    swiping.value = false  
    options?.afterEnd?.(e)  
  }  
  
  onMounted(() => {  
    if (!element.value) {  
      return  
    }  
    element.value.addEventListener('touchstart', onStart)  
    element.value.addEventListener('touchmove', onMove)  
    element.value.addEventListener('touchend', onEnd)  
  })  
  onUnmounted(() => {  
    if (!element.value) {  
      return  
    }  
    element.value.removeEventListener('touchstart', onStart)  
    element.value.removeEventListener('touchmove', onMove)  
    element.value.removeEventListener('touchend', onEnd)  
  })  
  return {  
    swiping,  
    direction,  
    distance,  
  }  
}
```

使用的时候

```tsx
const {swiping, direction} = useSwipe(refMain, {  
  beforeStart: e => e.preventDefault()  
})
const push = throttle((_direction: 'left' | 'right') => {  
  if (_direction === 'left' && currentPath.value < 4) {  
    router.push(`/welcome/${currentPath.value + 1}`)  
  } else if (_direction === 'right' && currentPath.value > 1) {  
    router.push(`/welcome/${currentPath.value - 1}`)  
  } else if (_direction === 'left' && currentPath.value === 4) {  
    router.push(`/`)  
  }  
}, 300)  
watchEffect(() => {  
  if (swiping.value && (direction.value === 'left' || direction.value === 'right')){  
    push(direction.value)  
  }  
})
<main ref={refMain}></main>
```

---
title: EyeDropper 对图片进行取色
date: 2024-09-25T11:17:33+08:00
updated: 2024-09-25T11:20:17+08:00
dg-publish: false
---

```js
btn.onclick = async () => {
  const dropper = new EyeDropper()
  try {
    const result = await dropper.open()
    console.log(result.sRGBHex)
  } catch {
    console.log('user canceled')
  }
}
```

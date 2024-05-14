---
title: 002-使用 Fetch-API 获取后端数据
date: 2024-03-07 20:52
updated: 2024-04-19 10:31
---

```js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
  history: createWebHistory(),
  routes: [
    //...
  ],
})
```

使用这种模式需要搭配服务器/后端使用，否则会出现 404

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

如果是在 Vercel 上可以添加配置文件 `vercel.json`

```json
{
  "rewrites": [{ "source": "/:path*", "destination": "/index.html" }]
}
```

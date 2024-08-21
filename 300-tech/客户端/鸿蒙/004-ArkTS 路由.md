---
title: 004-ArkTS 路由
date: 2024-06-19T02:30:00+08:00
updated: 2024-08-21T10:32:39+08:00
dg-publish: false
---

页面路由指在应用程序中实现不同页面之间的跳转和数据传递

页面栈：

- 点击跳转新页面压入栈
- 点击返回从栈中弹出
- 页面栈的最大容量上限为 32 个页面，使用 `router.clear()` 方法可以清空页面栈，释放内存
- Router 有两种页面跳转模式
	- `router.pushUrl()`：目标页不会替换当前页，而是压入页面栈，因此可以用 `router.back()` 返回当前页
	- `router.replaceUrl()`：目标页替换当前页，当前页会被销毁并释放资源，无法返回当前页（例如登录页跳转其他页）
- Router 有两种页面实例模式
	- Standard：标准实例模式，每次跳转都会新建一个目标页并压入栈顶，默认是这种模式
	- Single：单实例模式，如果目标页已经在栈中，则离栈顶最近的同 Url 页面会被移动到栈顶并重新加载（例如商品间切换就可以 pushUrl 搭配单实例模式）

使用

导入 HarmonyOS 提供的 Router 模块

```ts
import router from '@ohos.router'
```

然后利用 router 实现跳转、返回等操作

```ts
// 跳转到指定页面
router.pushUrl({
    url: 'pages/Todos',
    params: {id: 1} // 可选 传递的参数
  },
  router.RouterMode.Single,  // 页面模式 RouterMode 枚举
  err => { // 异常相应回调函数
    if (err) {
      console.log('路由跳转失败')
    }
  }
)

// 获取传递过来的参数
params: any = router.getParams()

// 返回上一页
router.back()

// 返回到指定页
router.back({
  url: 'pages/Index',
  params: {id: 10}
})
```

错误码：

```ts
100001 内部错误，可能是渲染失败
100002 路由地址错误
100003 路由栈中页面超过 32
```

如果是手动创建的文件，需要 `src/main/resources/base/profile/main_pages.json` 添加路由，如果是新建 Page 会自动添加

```ts
{
  "src": [
    "pages/Index",
    "pages/Goods",
    "pages/Picture",
    "pages/Todos"
  ]
}
```

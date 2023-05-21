进入官网：[Home v6.4.2 | React Router](https://reactrouter.com/en/main)
React 的文档最好先完整地看完一遍再开始
我们现在先 进入 Tutorial 中，查看教程

## 安装 Router
之前我们已经用脚手架创建好了项目，所以这句不用运行
```zsh
npm create vite@latest name-of-your-project -- --template react
# follow prompts
cd <your new project directory>
npm run dev
```

现在可以直接安装
```zsh
npm install react-router-dom
# 可以指定版本
npm install react-router-dom@6.4.2
```

## 初始化 Router
在 `main.tsx` 引入
```tsx
import { Route} from "react-router-dom"; // 可以只引入 Route
```

会飘红，提示应该放在 App 之前，我们使用了 `ESLint` 直接保存它会自动调整顺序
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221023092805.png)

提示没有使用，我们在控制台输出一下
```tsx
window.console.log(Route)
```

控制台中有输出
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221023093104.png)

接着再引用路由(路由就是根据用户访问的路径展示视图 核心是个表 所以需要创建表)
首先要创建一个根路由
Router 就是路由器，是完成那个表的人
在 `main.tsx` 中创建路由器
```tsx
const router = createBrowserRouter([
  {
    path: "/",
    element: <div>Hello world!</div>,
  },
])
```

会提示找不到需要引入，点击快速修复，更新导入并且把不需要的 Route 删除：
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221023094233.png)

在 reate 中使用
```tsx
<RouterProvider router={router} />
```

更新导入，删除不需要的导入，并自动排序
```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { RouterProvider, createBrowserRouter } from 'react-router-dom'

const router = createBrowserRouter([
  {
    path: '/',
    element: <div>root</div>,
  },
])

const div = document.getElementById('root') as HTMLElement
const root = ReactDOM.createRoot(div)

root.render(
  <React.StrictMode>
    <RouterProvider router={router} />
  </React.StrictMode>,
)
```

试下其他路径
```tsx
const router = createBrowserRouter([ // 这个是 history 模式，createHashRouter 哈希模式
  {
    path: '/',
    element: <div>root</div>,
  },
  {
    path: '/1',
    element: <div>root</div>,
  },
])
```

在地址栏中域名后加 `/1`，可以正常显示
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221023095315.png)
随便乱写
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221023095428.png)

> Ctrl+S 自动引入和调整，如果还是显示有错，再点击💡，手动解决
> 每完成一步就提交

## 处理 404
在根路由中加入 `errorElement` 字段
```tsx
{
	path: '/',
	element: <div>root</div>,
	errorElement: <div>你访问的页面不存在</div>, // 在这里添加了这个
},
```

自己定义 404 页面，error 类型这里暂定为 `any`
```tsx
import { useRouteError } from 'react-router-dom'
export const ErrorPage: React.FC = () => {
	const error: any = useRouteError()
	console.error(error)
	return (
		<div id="error-page">
			<h1>Oops!</h1>
			<p>Sorry, an unexpected error has occurred.</p>
			<p>
				<i>{error.statusText || error.message}</i>
			</p>
		</div>
	)
}
```

之后在 `main.tsx` 中引入并使用它
```tsx
import React from 'react'
import ReactDOM from 'react-dom/client'
import { RouterProvider, createBrowserRouter } from 'react-router-dom'
import { ErrorPage } from './components/ErrorPage' // 引入
const router = createBrowserRouter([
	{
		path: '/',
		element: <div>root</div>,
		errorElement: <ErrorPage />, // 使用
	},
	{
		path: '/1',
		element: <div>page1</div>,
	},
])
const div = document.getElementById('root') as HTMLElement
const root = ReactDOM.createRoot(div)
root.render(
	<React.StrictMode>
		<RouterProvider router={router} />
	</React.StrictMode>,
)
```

在浏览器中显示结果（它不只是能处理 404，还能处理其他错误）
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221023213722.png)


## 嵌套路由

写 `/子路由名字` 或者 `children` 中加入子路由信息两种都可以，React 没有严格规定
我们想让页面中显示子路由的内容，需要加上插槽 `<Outlet />` （自动引入）
需要让根路由显示子路由的路径，但是子路由中只显示它自己，则需在 `children` 第一行加上 `index: true`，其他下面的 `index:false` 可以不加
要是没有其他内容，就可以去掉 `<div>`（加这个方便加样式），只留 `<Outlet />`，只有一个 ``<Outlet />`` 就可以不写根路由的 `element` 字段
也可以单独把一个子路由给写出来（写出来就不显示 root），两个地方同时都写会用上面那个（优先匹配先出现的）
加不加 `/` 有没有影响
```tsx
const router = createBrowserRouter([
	{
	path: '/',
	element: <div>root <Outlet /></div>, // 可以不加 div，如果只需要 Outlet 那么这一行可以不要 element: <Outlet />
	errorElement: <ErrorPage />,
		children: [
			{ index: true, element: <div>请选择：1 2 3</div> }, // 路径为/
			{ path: '1', element: <div>1</div> }, // 路径为/1
			{ path: '2', element: <div>2</div> }, // 路径为/2
			{ path: '/3', element: <div>3</div> }, // 路径为/3，可以加 /，但是没必要
			// { path: '4', element: <div>4</div> }, // 会显示为 root 4 这里写了下面的就不生效
		],
	},
	{
		path: '/4',
		element: <div>page4</div>, // 显示 page4
	},
])
```


## 实现欢迎页面
添加四个欢迎页面
```tsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <Outlet />,
    errorElement: <RedirectToFeature1 />,
    children: [
      { index: true, element: <div>空</div> },
      {
        path: 'feature',
        element: <Outlet />,
        children: [
          { index: true, element: <div>空</div> },
          { path: '1', element: <div>1</div> },
          { path: '2', element: <div>2</div> },
          { path: '3', element: <div>3</div> },
          { path: '4', element: <div>4</div> },
        ],
      },
    ],
  },
])
```

不想让他们进入到不存在的页面，在 errorElement 中进行重定向
```tsx
import { useEffect } from 'react'
import { useNavigate } from 'react-router-dom'
export const RedirectToFeature1: React.FC = () => {
  const nav = useNavigate()
  useEffect(() => {
    nav('feature/1')
  }, [])
  return null
}
```
点击切换
```tsx
const router = createBrowserRouter([
  {
    path: '/',
    element: <Outlet />,
    errorElement: <RedirectToFeature1 />,
    children: [
      { index: true, element: <div>空</div> },
      {
        path: 'feature',
        element: <Outlet />,
        children: [
          { index: true, element: <div>空</div> },
          {
            path: '1',
            element: (
              <div>1 <NavLink to="/feature/2">下一页</NavLink></div>
            ),
          },
          {
            path: '2',
            element: (
              <div>2 <NavLink to="/feature/3">下一页</NavLink></div>
            ),
          },
          {
            path: '3',
            element: (
              <div>3 <NavLink to="/feature/4">下一页</NavLink></div>
            ),
          },
          {
            path: '4',
            element: (
              <div>4 <NavLink to="/home">开始记账</NavLink></div>
            ),
          },
        ],
      },
    ],
  },
])
```


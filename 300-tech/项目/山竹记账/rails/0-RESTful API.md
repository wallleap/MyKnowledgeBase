后端返回 API 有多种风格

## REST 是什么？
REpresentational State Transfer，是一种网络软件**架构风格**
- 不是标准、不是协议、不是接口，只是一种风格
- Roy 于 2000 年在自己博士论文中提到此术语
- Roy 曾参与撰写 HTTP 规格文档

## 怎么做？
- 以**资源**为中心
- 充分利用**HTTP 现有功能**，如动词、状态码、头部字段
- [GitHub API](https://docs.github.com/rest) 就比较符合 REST，值得学习

## 举例
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221015001931.png)
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221015002146.png)
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221015003149.png)
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221015003204.png)


- 状态码：https://developer.mozilla.com/en-US/docs/Web/HTTP/Status
- 创建用 POST 就好了，PUT 会传一个 ID
- 金钱最好用最小的单位（分），这样 JS  的加减乘除精度问题就可以解决了（或者用插件）
- json、urlencoded、xml 都可以，后端坚持使用一种就行

## REST 风格总结
- 尽量以资源为中心（名词）
- 尽量使用 HTTP 现有功能
- 可以适当违反规则（例如获取 ……/search/hi）

通俗点：
- 看到**路径**就知道请求什么东西
- 看到**动词**就知道是什么操作
- 看到**状态码**就知道结果是什么
	- 200——成功    201——创建成功
	- 404——未找到  403——没有权限     401——未登录
	- 402——无法处理，参数有问题    402——需付费
	- 412——不满足前提条件    429——请求太频繁
	- 400——其他所有错误，详细原因可以放在 body 里
```json
{
	"error_code": 4001,
	“error_msg": "err_more_money",
	"reason": "need_more_money"
}
```

## 为什么不喜欢 REST
![](https://cdn.wallleap.cn/img/pic/illustrtion/20221015004505.png)

选择风格和后端框架相契合

>符合 REST 风格的 API 就叫做 RESTful API
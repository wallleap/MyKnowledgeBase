---
title: 常用 API
date: 2024-03-11T11:45:00+08:00
updated: 2024-09-06T10:59:02+08:00
dg-publish: false
---

FREE API：<https://www.free-api.com/>

## 获取 IP

[myip.ipip.net/json](https://myip.ipip.net/json)

<https://api.ip.sb/geoip>

<https://www.ip.cn/api/index?ip&type=0>

<http://ip-api.com/json>

<https://ipapi.co/json/>

<https://ip.useragentinfo.com/json>

## 翻译

<https://clients5.google.com/translate_a/tclient=dict-chrome-ex&sl=auto&tl=zh-cn&q=word>

```
tl翻译类型 sl语言识别类型
zh-CN：中文  
en：英语  
ja：日语  
ko：韩语  
fr：法语  
de：德语  
ru：俄语  
es：西班牙语  
pt：葡萄牙语  
it：意大利语  
vi：越南语  
id：印尼语  
ar：阿拉伯语  
nl：荷兰语  
  
q=>翻译内容
```

## 假数据

[JSONPlaceholder - Free Fake REST API (typicode.com)](https://jsonplaceholder.typicode.com/)

<https://jsonplaceholder.typicode.com/todos/1>

## 图片

<https://devtool.tech/api/placeholder/800/300>

<https://picsum.photos/200/300>

<https://source.unsplash.com/random/800x600>

## 返回数据

status 就直接在响应头里，不放到响应内容中

```json
{
  "code": 200001,
  "msg": "描述信息",
  "data": {}
}
```

- code 根据状态码，后面加数字定好规范
	- 200：请求成功
	- 201：资源创建成功
		- 201001 用户创建成功
		- 201002 session 创建成功
	- 400：请求参数错误
	- 401：未授权访问
	- 403：表示禁止访问资源。
	- 404：表示未找到资源。
	- 500：表示服务器内部错误。
- msg
	- 附加描述信息
- data
	- 成功放资源
	- 失败放错误

---
title: 常用 API
date: 2024-03-11T11:45:00+08:00
updated: 2024-10-21T02:58:32+08:00
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

## 链接

### 路径

资源

### 参数

`url?q=keyword&sort=created&order=desc&per_page=20&page=1`

### 请求头

```
Accept: 'application/json'
Authentication: 'Bearer token-value'
```

### 请求体

## 动作

- GET：获取资源（单个/多个）
- POST：创建资源
- PATCH：修改资源的某个属性
- PUT：替换资源或资源的某个集合
- DELETE：删除资源

## 返回数据

status 就直接在响应头里，不放到响应内容中

```json
{
  "code": 200001,
  "msg": "描述信息",
  "data": {},
  "error": null
}
```

- code 根据状态码，后面加数字定好规范
	- 200：请求成功
	- 201：资源创建成功
		- 201001 用户创建成功
		- 201002 session 创建成功
	- 400：请求参数错误
	- 401：未授权访问
		- 401001 token invalid 错误或没有登录
		- 401002 过期
	- 403：表示禁止访问资源。
	- 404：表示未找到资源。
	- 500：表示服务器内部错误。
- msg
	- 附加描述信息
- data
	- 资源
- error
	- 失败错误

例如

**登录**

```
POST /api/v1/login {"username":"admin","password":"123456","captcha":"741420","captchaId":"DFs71CjyzfA7o4SohrOS","openCaptcha":true}

{
    "data": {
        "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkYXRhIjpbeyJ1c2VyTmFtZSI6IlNveWJlYW4ifV0sImlhdCI6MTY5ODQ4NDg2MywiZXhwIjoxNzMwMDQ0Nzk5LCJhdWQiOiJzb3liZWFuLWFkbWluIiwiaXNzIjoiU295YmVhbiIsInN1YiI6IlNveWJlYW4ifQ._w5wmPm6HVJc5fzkSrd_j-92d5PBRzWUfnrTF1bAmfk",
        "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJkYXRhIjpbeyJ1c2VyTmFtZSI6IlNveWJlYW4ifV0sImlhdCI6MTY5ODQ4NDg4MSwiZXhwIjoxNzYxNTgwNzk5LCJhdWQiOiJzb3liZWFuLWFkbWluIiwiaXNzIjoiU295YmVhbiIsInN1YiI6IlNveWJlYW4ifQ.7dmgo1syEwEV4vaBf9k2oaxU6IZVgD2Ls7JK1p27STE"
    },
    "code": "200001",
    "msg": "登录成功",
    "error": null
}
```

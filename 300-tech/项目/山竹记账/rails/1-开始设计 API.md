使用 Rails 提供的强大工具

## 发送验证码
- 资源：validation_codes（能不能有多个决定是否加 s）
- 动作：create(POST)
- 状态码：200 | 201 | 422（需要邮箱/手机号 不满足要求） | 429（请求太频繁）

## 登入登出
- 资源：session
- 动作：create | destroy(DELETE)
- 状态码：200 | 422

## 当前用户
- 资源：me
- 动作：show(GET)

## 记账数据
- 资源：items
- 动作：create | update | show | index | destroy
	- update 对应 PATCH，表示部分更新
	- show 对应 GET /items/:id，用来展示一条记账
	- index 对应 GET /items?since=2022-01-01&before=2023-01-01
	- destroy 对应 DELETE，表示删除，一般为软删除（数据很重要，只是设置一个字段为 isDestroy 为 true，判断已经删除）

## 标签
- 资源：tags
- 动作：create | update | show | index | destroy
	- index 是展示所有的

## 打标签
- 资源：taggings（动词的名词形式）
- 动作：create | index | destroy
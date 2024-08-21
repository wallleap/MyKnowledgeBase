---
title: 024-Rails Security
date: 2024-08-09T06:03:00+08:00
updated: 2024-08-21T10:32:31+08:00
dg-publish: false
---

## Rails 安全防护措施

### session 和 cookie

- Cookie based session
- Data in cookie

id 等敏感数据不要存在 cookie，session 现在 Rails 已经加密的

使用 https 而不是 http，不要明文传输，防止中间人攻击

### User input

- CSRF - RESTful / csrf_token
- XSS - h / sanitize
- File download / upload
- System command - `system()`
- HTTP Request methods
- Redirection
- Regular Expression - `/^$/` vs. `/\A\Z/`

## 数据库安全

- Mass assignment - `attr_accessible`、`attr_protected` / `params.require`
- SQL injection - use native method / sanitize sql
- Unscoped finds
- Batch update - `update_all`

例如，只有自己才能编辑自己的博客，如果直接 find 在 URL 中修改是能直接改别人的

```rb
def edit
  # @blog = Blog.find(params[:id])
  @blog = current_user.blogs.find(params[:id])
  render action: :new
end
```

## Rails 安全相关设置

- Running environments
- Passwords
- Filter logs parameters - log / backtrace
- Routes - default routes

## Rails 和 libs 的 bug

需要经常关注他们的公告、论坛，了解版本中是否存在 bug，及时修复

安全方面也可以用下这个 Gem: [https://github.com/presidentbeef/brakeman](https://github.com/presidentbeef/brakeman)

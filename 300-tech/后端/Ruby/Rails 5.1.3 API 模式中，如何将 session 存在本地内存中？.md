---
title: Rails 5.1.3 API 模式中，如何将 session 存在本地内存中？
date: 2024-10-21T11:19:15+08:00
updated: 2024-10-21T11:19:40+08:00
dg-publish: false
---

API 模式应该默认没加载 session 中间件。

Session 需要 cookies，API 不假设客户端支持 cookie，所以应该用别的机制保存用户状态。

---

想将 session 的信息存在本地缓存中，自己也新建了

```
config/initializers/session_store.rb 
```

并填写了内容

```
Rails.application.config.session_store :cache_store, key: '_test_api_session'
```

但是我在使用中 设置了 session["test"] = 123 后，在其他类的方法里 通过 session["test"] 无法获取到 123，在查看 session 为空。

不知道是哪里设置的不对还是我理解的不对。

求解答。不知道问题有没有描述清楚....

----------------------------update ---------------------------------

找到了解决办法。上午找了一上午没找到，这会一会就找到了。

参考了 [https://stackoverflow.com/questions/15342710/adding-cookie-session-store-back-to-rails-api-app](https://stackoverflow.com/questions/15342710/adding-cookie-session-store-back-to-rails-api-app) 中的解决办法。

原因 是因为 Rails API-only 模式 并不支持 cooke 与 session 这些，需要手动添加支持。

添加方法是在 config/application.rb 中追加 下面这两段

```
config.middleware.use ActionDispatch::Cookies                                                                        
config.middleware.use ActionDispatch::Session::CookieStore, key: '_test_api_key'
```

然后根据我上面说的，创建那个文件指定是哪种方式，默认是 cooke 存储 session。如果你有别的需要，可以对其修改，支持本地、Active_record 以及 memcache 等的存储。

---
title: Rails 默认 Session 的存储方式：CookieStore
date: 2024-08-16T09:55:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
source: //ruby-china.org/topics/39083
---

> This cookie-based session store is the Rails default. It is dramatically faster than the alternatives.

[https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html](https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html)

或许很多人不知道这个默认规则！

面试的时候不熟悉 Rails 的人会想当然的回答 Session 是存在服务端的，再一问存服务端哪里的，基本就蒙了 😇，有说 Redis，更有说进程内存里面。

---

实际上这个是一个安全的做法，[CookieStore](https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html) 会基于 `secure_key_base` 对 Session 内容进行 `aes-256-cbc` 加密，并将最终加密以 Base64 的结果返回作为 Cookie。

于是我们会看到 `__your_app_session` 这样的 Cookie 普遍在 Rails 项目里面存在：

[](https://l.ruby-china.com/photo/2019/ee997988-2fa2-43e7-8b6b-be0fcd34ccde.png!large)

[![](https://l.ruby-china.com/photo/2019/ee997988-2fa2-43e7-8b6b-be0fcd34ccde.png!large)

](<https://l.ruby-china.com/photo/2019/ee997988-2fa2-43e7-8b6b-be0fcd34ccde.png!large>)

Rails 哲学里面提倡最佳实践，CookieStore 应该算是一个，不需要依赖服务端的持久化数据库（Redis / Memcache / MySQL）既可实现。

## CookieStore 的优点

- 简单有效，无需额外的后端存储，开箱即用；
- 相对与其他存储方式，它很快，只需解密，无 IO 请求；
- 因为 Session 加密存储在用户浏览器，所以不回因为后端存储丢失而导致 Session 失效；
- 我们依然可以通过改变 `secure_key_base` 的方式来使之前的 Session 失效；

## 缺点

- 浏览器 Cookie 限制，加密过后的内容不能超过 4K，所以我们不能在 Session 里面放过多内容；
- 如果你的静态资源没有独立域名的话，静态页面的请求 Header 也会多余带这个 Session 的值；
- Session 存储过大内容到了客户端，会导致用户访问服务错误，切难以被发现；
- 如果那种验证 code 存 Session 里面，会有重现攻击（Replay attacks）风险 🚨，参考：[RuCaptcha 说明](https://github.com/huacnlee/rucaptcha#usage)；

---

## 一些提示

- `flash` 实际上也是存储在 Session 里面的；
- 尽量在 session 里面存 user_id 即可，别把整个 user 对象放进去；
- 因为客户端实际上能拿到 Session 也可以通过请求 set-cookie header 的方式修改 Cookie，所以 `secure_key_base` 一定要保密，一旦发现泄漏，尽快改掉；
- 改掉 `secure_key_base` 的副作用是已经有 Session 会因为无法失败而被当成失效的，表现上看就是用户需要重新登录；
- 避免敏感数据存 Session 里面，以免被重现攻击；

> 🔖如果你掌控不了，或目前项目已有敏感数据在 Session 里面，图省事你可以换成 [CacheStore](https://api.rubyonrails.org/classes/ActionDispatch/Session/CacheStore.html) (Rails >= 5.2)，Cache 存哪里 Session 就存哪里。

## 参考阅读

- [Rails Guides / Security / Sessions](https://guides.rubyonrails.org/security.html#sessions)
- [Rails API / CookieStore](https://api.rubyonrails.org/classes/ActionDispatch/Session/CookieStore.html)
- [MessageEncryptor 文档](https://api.rubyonrails.org/classes/ActiveSupport/MessageEncryptor.html)
- [CookieStore Sessions 重现攻击](https://guides.rubyonrails.org/security.html#replay-attacks-for-cookiestore-sessions)
- [Gist - 如何手工解密 Session](https://gist.github.com/tonytonyjan/d71f040fe1085dfcf1d4)

---

一直以为这个默认的

他就是默认的，但知道 Cookie-based Session 的人凤毛菱角，Rails 开发知道的还算多一些

---

记得刚开始接触 rails 的时候就遇到过 session 超出大小限制的情况，因为当时将很多信息都存在 session 里，后来改用 cachestore 了，那时还是个刚入行的新人。

感觉一般 CookieStore 就够用了

如果有以下情况，我会考虑换到 redis

1. 4KB 不够用
2. 想控制失效的时间
3. 无法承受 Replay attacks 带来的风险

---

如果要使某个用户的 session 失效，怎么办？

让 session 失效这个想法可以换成让 session 内保存的内容失效

---

有个最大的优点没提哟：天然支持分布式

---

Cookie-based Session 就是客户端的 session？那就是 cookie 啊。。。

---

php 的就是默认的服务器端的/tmp 下文件中的吧！我一直以为 rails 的也是在服务端的文件中呢！原来是在客户端的 cookie 里面保存的啊！

---

CookieStore 儲存方式 在 多主機 or multi process 的狀態下，運作可以運作的很好

尤其是需要應付高並發的情境下 ( 如：每年的 1111 / 1212 .. etc ) 除了可減少主機內部網路負擔，也可以避免伺服器之間，可能會有過度依賴的問題

因為你加開的主機越多，只要有負責處理特定服務的主機掛掉，整體系統就會垮掉無法使用 所以你必須在硬體主機上 或 架構上做一些調整，以便成本可以控制得住，又可以達成相同的目的

就以 database 跟 存放 session 主機來看，我一定要讓 database 要活著，因為這不能掛 但有關 session 的存放我就可以另尋方案，像 CookieStore 的方式我可能就會考慮

[](https://l.ruby-china.com/photo/2019/4c75e6a3-9725-40b9-bb6a-f9e9118a0434.png!large)

[![](https://l.ruby-china.com/photo/2019/4c75e6a3-9725-40b9-bb6a-f9e9118a0434.png!large)

](<https://l.ruby-china.com/photo/2019/4c75e6a3-9725-40b9-bb6a-f9e9118a0434.png!large>)

我個人重新審視使用 CookieStore 時，是因為 Elixir 的 Phoenix Framework ( 狀況需要 ) 仔細研究了一下，他們也是採用加密的 cookie，所以非常容易實現高並發， 若採 redis、memcache 的方案，恐怕速度未必拉得起來，而且可能會有東西先掛掉

[](https://l.ruby-china.com/photo/2019/6053c877-e0b5-443d-8494-a9f6ad409638.png!large)

[![](https://l.ruby-china.com/photo/2019/6053c877-e0b5-443d-8494-a9f6ad409638.png!large)

](<https://l.ruby-china.com/photo/2019/6053c877-e0b5-443d-8494-a9f6ad409638.png!large>)

以前 new 一個 rails 的專案都是直接將 cookie store 換掉 ( 過去的習慣 ) 直到平台的架構需要 ( 提升性能 ) 才覺得這個設計好像也沒有什麼問題。

但我在研究的過程中，有發現一個安全上的缺點

第一，decrypt 已加密的 cookie 腳本到處找得到 第二，相關的 salt 值意外的很簡陋

根據 [https://edgeguides.rubyonrails.org/configuring.html](https://edgeguides.rubyonrails.org/configuring.html) 的描述 用來為 cookie 做加密的兩組 salt 值 signed_cookie_salt、encrypted_cookie_salt 也似乎不曾在專案的設定裡

```
Rails.application.config.action_dispatch.encrypted_cookie_salt
# 結果為 "encrypted cookie"

Rails.application.config.action_dispatch.encrypted_signed_cookie_salt
# 結果為 "signed encrypted cookie"
```

換句話說，只要你的系統專案、平台，開發成員離職 or 退出，若他們仍記著 secret key

就可透過以下的方式解回 cookie

```
# 可在網路搜尋到 Rails 4 ~ 5.x 的 session cookie 解回的方法
def verify_and_decrypt_session_cookie(cookie, secret_key_base)
  cookie = CGI::unescape(cookie)
  salt = "encrypted cookie"
  signed_salt = 'signed encrypted cookie'
  key_generator = ActiveSupport::KeyGenerator.new(secret_key_base, iterations: 1000)
  secret = key_generator.generate_key(salt)[0, 32]
  sign_secret = key_generator.generate_key(signed_salt)
  encryptor = ActiveSupport::MessageEncryptor.new(secret, sign_secret, serializer: JSON)

  encryptor.decrypt_and_verify(cookie)
end
```

既然可以解回來，是不是有可能根據 Rails 4、5、6 的版本做出破解，甚至將改過的資料送回 原本的伺服器， 以便偽造資料，我沒有時間可以驗證 但我有印象根據 rails 的 changelog 上，有發現他們有改進 CookieStore 的一些安全上的缺陷

但無論如何，我認為還是要再更新這兩組 salt，定時更換會比較妥當

( 這兩組設定要另外 key 上去，沒有在註解裡的樣子 )

```
Rails.application.configure do

  # 設定 兩組 salt 給 encrypted cookie，salt 的長度隨意，記得定時偶爾換就好
  config.action_dispatch.encrypted_cookie_salt = '26c8e554....'
  config.action_dispatch.encrypted_signed_cookie_salt = '098d7fdd....'

end
```

此外，為了讓 session data 無效，更換 secret_key_base 是可能極為冒險的

如果假設你的系統有套上 devise 作為會員登入系統的基礎

根據 devise 的文件的描述，原始碼 Devise::SecretKeyFinder 這個 module

他會抓 secret_key_base 來作為 忘記密碼、註冊認證信的連結使用，

若你為了 session 無效而更換下去，寄給網友的通知信，裡面的連結都將全數失效，無法使用。

所以，個人的淺見是，可以更換 encrypted_cookie_salt、encrypted_signed_cookie_salt 來達成目的可能會比較適當。

不過此部分的更換一定需要在離峰時間處理， 若在平常運作下，系統線上很多人得情況下更換，會產生驚人數量的 invalid csrf token 之類的 exception 記錄在主機上，有掛 New Relic 可能還會收到警報通知不停 QQ

以上，純屬看法跟經驗分享。

---

昨天面试了一个候选人，我也问了类似的问题，我说 session 能不能存在 cookie 里，对方立马回答怎么可能。

BTW，之前为了给女票解释 session 和 cookie 的区别，还搜到过斯坦福大学网站上面一篇 CS142 的讲稿，里边也是探讨 session 和 cookie 的关系，还特地以 Rails 为例做了讲解： [https://web.stanford.edu/~ouster/cgi-bin/cs142-fall10/lecture.php?topic=cookie](https://web.stanford.edu/~ouster/cgi-bin/cs142-fall10/lecture.php?topic=cookie)

我自己很多 Web 领域的知识，来自于对 Rails 框架本身的学习。

---

Elixir 的话，可以用 Mnesia 存，速度快，分布式，健壮，ACID。mnesia 的缺点是和 app 共享内存。

不过 Elixir 社区似乎不怎么待见 mnesia。

理论上 mnesia 要比 redis 快很多（有人比较过 ets，[ets versus redis benchmarks for a simple key value store](https://minhajuddin.com/2017/10/04/ets-versus-redis-benchmarks-for-a-simple-key-value-store/)）。

redis 再怎么说也是单线程。。。

> 补充

本地实际压了一下，4 core，mnesia 只比 redis 快了几倍，性能优势不明显。

用 dirty_read，性能会有上百倍的提升，但这样就没有一致性了。

另外 redis 有社区版，支持多线程，[https://ruby-china.org/topics/39083#reply-357150](https://ruby-china.org/topics/39083#reply-357150)

---

有多线程的了 [https://github.com/JohnSully/KeyDB](https://github.com/JohnSully/KeyDB)

哈，redis 竟然有多线程的版本了。

不过感觉 mnesia 应该会更快。因为 mnesia 是跑在应用里，本地读写，不用走网络，不用序列化，反序列化。 ets 的 hashtable 本神是可以被并发（不过这个并发数也不是很大）写不同 的 key 的，KeyDB 似乎是用 spinlock 锁了 table。

性能最重要的要看 lock 和 transaction 咋实现的。

> Transactions hold the lock for the duration of the EXEC command. Modules work in concert with the GIL which is only acquired when all server threads are paused. This maintains the atomicity guarantees modules expect.

KeyDB 有👆这么一句，不过没看懂。。。代码扫了眼，也没看懂。。。

---

是不是可以用 `reset_session`

[https://guides.rubyonrails.org/action_controller_overview.html#accessing-the-session](https://guides.rubyonrails.org/action_controller_overview.html#accessing-the-session)，文档里最后一句有写

> To reset the entire session, use `reset_session`.

---

我打脸了，本地（4 core）测了一下，mnesia 本地压测。和 redis 速度差不多。。。只是快了几倍。。。

除非 cpu 核心数特别多，或者几点特别多（也可能有问题），或者不需要好正一致性的情况，mnesia 在性能上才会有明显的优势。。。

---

只做缓存的话 memcache 呀 支持多线程

---

主要 mnesia 是 Erlang 自带的，不过大家对 mnesia 都不熟，这个东西学习成本还不低。。。

类似的产品有多。同样是 Erlang 实现，倒是 RabbitMQ 很有市场。

除了特殊场景，性能一般不会是瓶颈。。。

---

session 存在客户端那边可不是什么最佳实践。。。。。因为：优点可以忽略不计，缺点数不胜数 一句话，费劲。

session 何止能放在 cookie 里面，当成参数放在 URL 链接上都是可以的

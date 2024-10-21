---
title: 微信 UA 识别
date: 2024-09-14T05:33:40+08:00
updated: 2024-09-14T05:50:05+08:00
dg-publish: false
---

iPhone 通过微信内置浏览器访问网页时得到 User Agent 是：

```
Mozilla/5.0 (iPhone; CPU iPhone OS 6_1_3 like Mac OS X) AppleWebKit/536.26 (KHTML, like Gecko) Mobile/10B329 MicroMessenger/5.0.1
```

Android 通过微信内置浏览器访问网页时得到 User Agent 是：

```
Mozilla/5.0 (Linux; U; Android 2.3.6; zh-cn; GT-S5660 Build/GINGERBREAD) AppleWebKit/533.1 (KHTML, like Gecko) Version/4.0 Mobile Safari/533.1 MicroMessenger/4.5.255
```

判断代码

```
function is_weixin(){
	var ua = navigator.userAgent.toLowerCase();
	if(ua.match(/MicroMessenger/i)=="micromessenger") {
		return true;
 	} else {
		return false;
	}
}
```

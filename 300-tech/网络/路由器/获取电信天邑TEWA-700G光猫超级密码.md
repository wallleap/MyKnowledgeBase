---
title: 获取电信天邑TEWA-700G光猫超级密码
date: 2025-03-12T09:48:42+08:00
updated: 2025-03-12T13:26:28+08:00
dg-publish: false
---

网上花 28 块钱买了个天邑光猫，同城的（山东电信），需要修改光猫参数，无奈在网上搜解决方法，最终找到 telecomadmin 密码，写文章记录下。

山东电信光猫的超级用户是:telecomadmin  密码是 telecomadmin+8 位数字。

1、将 U 盘格式化为 FAT32 格式，插入光猫 USB 口

2、打开<http://192.168.1.1/>光猫登录页，使用光猫背面的账号密码登录，查看 U 盘是否被识别。如识别到，进行下一步。

3、打开<http://192.168.1.1:8080/login.html>，使用光猫背面的账号密码登录，选择管理 - 设备管理或用户管理

![](https://www.yangtengfei.com/wp-content/uploads/2022/03/1.png)

4、F12 打开开发工具，选择元素，Ctrl+F 搜索：sessionkey，找到 sessionkey=xxxx 后面的数字，如我的设备：（查询到 3 串，其中一串有用，另外 2 串网页是报错的，都尝试下，一般是注释里面的那一串）

![](https://www.yangtengfei.com/wp-content/uploads/2022/03/QQ%E5%9B%BE%E7%89%8720220307231258.png)

5、打开：<http://192.168.1.1:8080/usbbackup.cmd?action=backupeble&sessionKey=729136729> （729136729 换成自己的 sessionKey）

选择备份配置，就可以将配置保存在 U 盘了。（如果备份配置是灰色点不动，快速恢复选择**禁用**）

或在控制台输入

```js
usbsubarea = { 
selectedIndex: 0, 
value: "never_mind...",
	options:[ 
	 	{ value: "usb1_1" } 
	] 
}; 
btnApply();
```

![](https://www.yangtengfei.com/wp-content/uploads/2022/03/QQ%E5%9B%BE%E7%89%8720220307231429.png)

6、将 U 盘拔下来插电脑，拷贝出备份文件，使用 RouterPassView 打开即可查看到密码。（软件会报毒，网上自行下载）

或者下载并解压 [解密工具](https://pan.baidu.com/s/1lXgn4Fr8fwKm_5wUNbINtw) 提取码：tenk

<https://wallleap.lanzoue.com/ieegG2qcp9pa>

鼠标拖拽 .cfg 文件到 xor 解密工具上，即得到 .cfg.xml 结尾的解密后文件，之后直接使用记事本打开，搜索 `password`

![](https://www.yangtengfei.com/wp-content/uploads/2022/03/QQ%E5%9B%BE%E7%89%8720220307231731.png)

帐号：telecomadmin

密码：telecomadmin18538276

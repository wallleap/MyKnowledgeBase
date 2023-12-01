---
title: 前端加密与 crypto-js 和 JSEncrypt 的使用
date: 2023-08-07 09:42
updated: 2023-08-07 09:43
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - 前端加密与 crypto-js 和 JSEncrypt 的使用
rating: 1
tags:
  - JS
  - web
category: web
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
description: 文章描述
source: #
url: //myblog.wallleap.cn/post/1
---

携手创作，共同成长！这是我参与「掘金日新计划 · 8 月更文挑战」的第 2 天，[点击查看活动详情](https://juejin.cn/post/7123120819437322247 "https://juejin.cn/post/7123120819437322247")

在网站项目中，有时我们需要对传给后端的数据，比如 token 等进行加密处理。本文是对几种常见的前端加密方法，以及如何使用开源的加密库 crypto-js、JSEncrypt 来实现它们的分享。

## 单向散列函数

又称为消息摘要算法，是**不可逆**的加密算法，即对明文进行加密后，无法通过得到的密文还原回去得到明文。一般所谓的比如 MD5 破解，其实是不断的尝试用不同的明文进行加密，直到得到的加密结果一致。常见的单项散列函数有 MD5、SHA1、SHA256、SHA512 ，以及它们之前加上 Hmac（Keyed-hash message authentication codes） 后的 HmacMD5、HmacSHA1 等。下面以 MD5 为例重点介绍，其它几种则可以举一反三，不多赘述：

## MD5

### 简单介绍

MD5 长度固定，不论输入的内容有多少字节，最终输出结果都为 128 bit，即 16 字节。这也就解释了为什么 MD5 以及其它单向散列函数是不可逆的 —— 输出定长代表会有数据丢失。通常，我们可以用 16 进制字面值来表示它，每 4 bit 以 16 进制字面值显示，得到的是一个长度为 32 位的字符串。注意，MD5 等单向散列函数具有高度的离散性，意思是只要输入的明文不一样，得到的结果就完全不一样，哪怕是仅仅多了一个空格。

#### 使用场景

MD5 有下面几种使用场景：

- 可以用来做密码的保护。比如可以不直接存储用户的密码，而是存储密码的 MD5 结果。但是现在一般认为用 MD5 来做加密是不太安全的，更多是利用 MD5 的高度离散性特点，用它来做数字签名、完整性校验，云盘秒传等；
- 数字签名。我们可以在发布程序时同时发布其 MD5。这样，别人下载程序后自己计算一遍 MD5，一对比就知道程序是否被篡改过，比如植入了木马。
- 完整性校验。比如前端向后端传输一段非常大的数据，为了防止网络传输过程中丢失数据，可以在前端生成一段数据的 MD5 一同传给后端，这样后端接收完数据后再计算一次 MD5，就知道数据是否完整了。

### 使用 crypto-js 进行 MD5 加密

为了方便，我采用了在普通 html 页面直接引入 cdn 的方式来引入 crypto-js。

```
<script src="https://cdn.bootcdn.net/ajax/libs/crypto-js/4.1.1/crypto-js.min.js"></script>
```

引入后，我们就能得到 `CryptoJS` 这个对象，它包含了很多方法，打印结果如下图：
![](https://cdn.wallleap.cn/img/pic/illustration/202308070943575.png)
其中就定义了 `MD5` 方法和 `algo` 对象，借助它们，可以分别得到输入数据的 MD5 结果：

#### CryptoJS.MD5()

```
CryptoJS.MD5('2022JueJin').toString()
```

结果为 84231025843afb62d818bf4f21612051。“2022JueJin” 就是我们需要加密的明文数据，得到的结果需要转为字符串输出，不然会是一个对象，而不是 16 进制字面值显示的 32 位字符串（由 0 - 9 和 a - f 组成）。

- **WordArray**

传入 `CryptoJS.MD5()` 的参数除了字符串外，还可以是 CryptoJS 定义的一种叫做 WordArray 的数据类型。

```
const wordArray = CryptoJS.enc.Utf8.parse('2022JueJin')
CryptoJS.MD5(wordArray).toString()
```

上面这段代码将 utf8 字符串先转成了 WordArray 对象，再将其传给 `CryptoJS.MD5()`，最终得到的结果如果打印输出的话也是 84231025843afb62d818bf4f21612051。enc 可以看成是 Encode（编码） 的缩写。`Utf8` 还能换成 `Latin1`、`Hex` 和 `Base64` 等。如果想将一个 WordArray 对象转换回 utf8 字符，可以执行

```
CryptoJS.enc.Utf8.stringify(wordArray)
```

一般情况下，消息摘要算法得到的结果都是以 16 进制字面值表示，如果想要得到 Base64，可以将加密结果通过 `CryptoJS.enc.Base64.stringify()` 转换：

```
const base64 = CryptoJS.enc.Base64.stringify(CryptoJS.MD5('2022JueJin'))
console.log(base64) // hCMQJYQ6+2LYGL9PIWEgUQ==
```

#### algo

如果需要生成 MD5 的数据是个大文件，一般我们可以把大文件分为多段。采用下面的方式，先使用 `CryptoJS.algo.MD5.create()` 创建一个对象，命名为 hasher。然后将数据一段段的传入 `hasher.update()` 处理：

```
const hasher = CryptoJS.algo.MD5.create()
hasher.update('2022')
hasher.update('JueJin')
const hash = hasher.finalize()
console.log(hash.toString())
```

最后调用 `hasher.finalize()` 表示传输完成，`finalize()` 里也可以传入数据。打印得到的结果也是 84231025843afb62d818bf4f21612051。如果想清除之前的 update，可以调用 `hasher.reset()`。

上面介绍的单向散列函数严格来说并不是加密算法，更多是用于签名。项目中需要进行加密的时候，最好采用下面介绍的加密算法。它们的加密和解密过程是可逆的，分为对称加密和非对称加密。

## 对称加密算法

所谓对称，指的是加密和解密使用的是相同的秘钥，常见的有 DES、3DES 、RC4、RC5、RC6 和 AES。下面以我在公司最近的项目中使用的 AES 为重点进行介绍。

## AES

英文全称为 Advanced Encryption Standard，即高级加密标准的意思。它的推出，用于取代已经被证明不安全的 DES 算法。AES 属于分组加密算法，因为它会把传入的明文数据以 128 bit 为一组分别处理。其秘钥长度则可以是 128、192 和 256 bit。AES 或者说对称加密算法的优点是速度快，缺点就是不安全，因为网站上的代码和秘钥都是明文，别人只要得到了加密结果再结合秘钥就能得到加密的数据了。

### 使用 crypto-js 进行 AES 加密

#### 加密

我们将 “JueJin2022” 通过 AES 加密，得到的将是一个对象，我们需要通过 `toString()` 将其转成字符串输出，最终得到的是一个以 base64 编码的 “5yOOaUK1NSxVcRc8TA1fZw==”，代码如下：

```
const message = CryptoJS.enc.Utf8.parse('JueJin2022')
const secretPassphrase = CryptoJS.enc.Utf8.parse('0123456789asdfgh')
const iv = CryptoJS.enc.Utf8.parse('0123456789asdfgh')

const encrypted = CryptoJS.AES.encrypt(message, secretPassphrase, {
  mode: CryptoJS.mode.CBC,
  padding: CryptoJS.pad.Pkcs7,
  iv
}).toString()

console.log(encrypted)
```

`CryptoJS.AES.encrypt()` 可以传入 3 个参数： 第 1 个为需要加密的明文； 第 2 个是秘钥，长度可以是 128、192 或 256 bit； 第 3 个为一个配置对象，可以添加一些配置。常见的配置属性有：

- mode：加密模式。默认为 CBC，还支持且常用的是 ECB。CBC 模式需要偏移向量 iv，而 ECB 不需要。
- padding：填充方式。默认为 Pkcs7；
- iv：偏移向量 ；

注意，明文、秘钥和偏移向量一般先用诸如 `CryptoJS.enc.Utf8.parse()` 转成 WordArray 对象再传入，这样做得到结果与不转换直接传入是不一样的。

#### 解密

解密的写法和加密差不多，只是把 `encrypt` 方法名改为 `decrypt`，然后传入的第 1 个参数由明文替换为密文，最后将之前转换明文的方式传入 `toString()` 即可：

```
const secretPassphrase = CryptoJS.enc.Utf8.parse('0123456789asdfgh')
const iv = CryptoJS.enc.Utf8.parse('0123456789asdfgh')
const decrypted = CryptoJS.AES.decrypt(
  '5yOOaUK1NSxVcRc8TA1fZw==',
  secretPassphrase,
  {
    mode: CryptoJS.mode.CBC,
    padding: CryptoJS.pad.Pkcs7,
    iv: '0123456789asdfgh'
  }
).toString(CryptoJS.enc.Utf8)
console.log(decrypted) // JueJin2022
```

注：如果之前在加密时没有将明文进行 parse 而是直接传入的，那么在解密时，传入 `toString()` 的解析方式就是写默认的 `CryptoJS.enc.Utf8`。

## 非对称加密

所谓的非对称，即加密和解密用的不是同一个秘钥。比如用公钥加密，就要用私钥解密。非对称加密的安全性是要好于对称加密的，但是性能就比较差了。

## RSA

非对称加密算法中常用的就是 RSA 了。它是由在 MIT 工作的 3 个人于 1977 年提出，RSA 这个名字的由来便是取自他们 3 人的姓氏首字母。我们在访问 github 等远程 git 仓库时，如果是使用 SSH 协议，需要生成一对公私秘钥，就可以使用 RSA 算法。

### 使用 JSEncrypt 进行 RSA 加密

我们依旧是采用 cdn 方式直接在页面中引入 JSEncrypt 库：

```
<script src="https://cdn.bootcdn.net/ajax/libs/jsencrypt/3.2.1/jsencrypt.min.js"></script>
```

使用的代码非常简单。首先需要 `new` 一个实例对象出来，然后将通过 openssl 生成的公钥传给实例对象的 `setKey` 方法，之后只需要把要加密的明文传给实例的 `encrypt()` 进行加密即可：

```
const crypt = new JSEncrypt()
crypt.setKey('openssl 生成的公钥')
const text = 'JueJin2022'
const enc = crypt.encrypt(text)
console.log(enc)
```

生成的密文是一段 base64 格式的 1024 位 RSA 私钥。

### 使用 JSEncrypt 进行 RSA 解密

解密就是把私钥传给实例的 `setKey()`，之后把密文传给 `decrypt()` 进行解密即可：

```
const crypt = new JSEncrypt()
crypt.setKey('openssl 生成的私钥')
const enc = 密文
const dec = crypt.decrypt(enc)
console.log(dec)
```

注意，setKey 有 2 个别名： 如果传入的是私钥，可以用 `setPrivateKey()` 替换 `setKey()`； 如果传入的是公钥，可以用 `setPublicKey()` 替换 `setKey()`；

### OpenSSL

从上面的内容可知，JSEncrypt 的加解密过程需要用到 OpenSSL 来生成秘钥，OpenSSL 是一个开源的软件，它是对 SSL 协议的实现。能够用于生成证书、证书签名、生成秘钥和加解密等。比如我公司最近的项目有个需求是要在本地开发时，localhost 使用 https 协议，就有用到 openssl。

#### 安装

可以去 [slproweb.com/products/Wi…](https://link.juejin.cn/?target=http%3A%2F%2Fslproweb.com%2Fproducts%2FWin32OpenSSL.html "http://slproweb.com/products/Win32OpenSSL.html") 选择对应版本安装：
![](https://cdn.wallleap.cn/img/pic/illustration/202308070943576.png)
然后在环境变量中添加配置，例如我把 openssl 安装在了 D:\\OpenSSL-Win64，就将 D:\\OpenSSL-Win64\\bin 添加到 Path 中：
![](https://cdn.wallleap.cn/img/pic/illustration/202308070943577.png)

#### 生成私钥

我们可以在想要保存秘钥的文件夹启动命令行工具，并输入以下命令生成秘钥文件：

```
openssl genrsa -out rsa_1024_priv.pem 1024
```

- genrsa： 生成 RSA 私有密钥；
- \-out：生成的密钥文件，后面配置的是我们生成的密钥文件的名字，可从中提取公钥；
- 1024：生成的秘钥长度为 1024 bit；

生成的秘钥文件如下：
![](https://cdn.wallleap.cn/img/pic/illustration/202308070943578.png)
可以通过 `cat rsa_1024_priv.pem` 查看秘钥内容，然后复制粘贴给上面的 `crypt.setKey()` 。 注意：秘钥必须写成一行以 `-----BEGIN PRIVATE KEY-----` 开头，以 `-----END PRIVATE KEY-----` 结尾。像下面这样：

```
crypt.setKey(
  '-----BEGIN PRIVATE KEY-----MIICdFCQBj...中间省略...D3t4NbK1bqMA=-----END PRIVATE KEY-----'
)
```

#### 生成公钥

生成上面的私钥对应的公钥的命令为

```
openssl rsa -pubout -in rsa_1024_priv.pem -out rsa_1024_pub.pem
```

- rsa：处理 RSA 密钥的格式转换等问题；
- \-pubout：指定输出文件是公钥；
- \-in：输入文件，也就是我们上面生成的私钥文件；
- \-out：输出文件，也就是是我们要生成的公钥文件；

查看同样是使用 `cat` 命令。

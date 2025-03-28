---
title: 020-使用 Nodemailer 发送邮件
date: 2025-03-21T10:57:15+08:00
updated: 2025-03-24T15:41:18+08:00
dg-publish: false
---

安装

```sh
npm install nodemailer
```

`.env` 中添加内容，到企业邮箱申请一个授权码，填入 PASS，USER 为邮箱地址

```
MAILER_HOST=smtp.qiye.aliyun.com
MAILER_PORT=465
MAILER_SECURE=true
MAILER_USER=
MAILER_PASS=
```

`utils/mail.js`

```js
const nodemailer = require('nodemailer')

const transporter = nodemailer.createTransport({
  host: process.env.MAILER_HOST,
  port: process.env.MAILER_PORT,
  secure: process.env.MAILER_SECURE,
  auth: {
    user: process.env.MAILER_USER,
    pass: process.env.MAILER_PASS
  }
})

/**
 * 发送邮件
 * @param {*} email 对方的邮箱
 * @param {*} subject 邮件标题
 * @param {*} html 邮件内容
 */
const sendMail = async (email, subject, html) => {
  await transporter.sendMail({
    from: process.env.MAILER_USER,
    to: email,
    subject,
    html
  })
}

module.exports = sendMail
```

`routes/auth.js` 用户注册成功后发送邮件

```js
const html = `
	<div style="font-family: Arial, sans-serif; line-height: 1.6; color: #333;">
		<h2 style="background-color: #4CAF50; color: white; padding: 10px; text-align: center;">注册成功</h2>
		<div style="padding: 20px; border: 1px solid #ddd; border-radius: 5px;">
			<p>亲爱的 ${user.username},</p>
			<p>感谢您注册我们的服务！您的账号已经成功创建。</p>
			<p>以下是您的注册信息：</p>
			<ul>
				<li><strong>邮箱:</strong> ${user.email}</li>
				<li><strong>用户名:</strong> ${user.username}</li>
				<li><strong>昵称:</strong> ${user.nickname}</li>
			</ul>
			<p>请妥善保管您的账号信息。</p>
			<p>祝您使用愉快！</p>
		</div>
	</div>
`
sendMail(user.email, '注册成功', html)
```

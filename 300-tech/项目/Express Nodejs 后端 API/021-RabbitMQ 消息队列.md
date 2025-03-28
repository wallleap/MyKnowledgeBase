---
title: 021-RabbitMQ 消息队列
date: 2025-03-21T10:57:27+08:00
updated: 2025-03-25T14:21:21+08:00
dg-publish: false
---

用户注册完成之后发送邮件，邮件发送完成之后才返回内容，但并不需要等待发送完成再返回

项目中经常要在一些不需要等待成功的地方使用消息队列，例如发送邮件、发送短信、应用内通知、文件处理、数据分析与报告生成、订单处理、秒杀等

## Docker 安装 RabbitMQ

`docker-compose.yml`

```yml
services:
  mysql:
    image: hub.vonne.me/mysql:8.3.0
    # 下面这行是为了解决mysql8.0.23版本之后，默认的编码为utf8mb4，而express-admin-ui中默认使用的是utf8编码，所以需要修改为utf8mb4编码
    command: mysqld --character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
    environment:
      - MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} # 设置 root 用户的密码
      - MYSQL_LOWER_CASE_TABLE_NAMES=0  # 默认表名不区分大小写，设置为0，则区分大小写
    ports:
      - "12306:3306"  # 映射端口，前面端口是宿主机端口，后面端口是容器端口
    volumes:
        - ./data/mysql:/var/lib/mysql  # 数据持久化
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 3
    profiles:
      - local
      - server

  redis:
    image: hub.vonne.me/redis:7.4
    ports:
      - "26379:6379"
    volumes:
      - ./data/redis:/data
    profiles:
      - local
      - server

  rabbitmq:
    image: hub.vonne.me/rabbitmq:4.0-management
    ports:
      - "5672:5672"
      - "15672:15672"
    volumes:
      - ./data/rabbitmq:/var/lib/rabbitmq
    environment:
      RABBITMQ_DEFAULT_USER: ${RABBITMQ_USER}
      RABBITMQ_DEFAULT_PASS: ${RABBITMQ_PASS}
    profiles:
      - local
      - server
      
  node-app:
    # 构建镜像，使用当前目录下的 Dockerfile
    build:
      context: .
      args:
        NODE_ENV: docker
    # 容器名称
    container_name: node-app-container
    # 端口映射，将容器的 3001 端口映射到主机的 3300 端口
    ports:
      - "3300:3001"
    # 重启策略，始终重启
    restart: always
    # 依赖于 mysql 服务
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_started
    profiles:
      - server
```

运行命令启动服务

```sh
docker compose --profile local up -d
```

## 管理后台

访问 <http://127.0.0.1:15672>，使用环境变量中配置的用户和密码进行登录

## 测试

安装

```sh
npm intsall amqplib
```

在 `public` 中新建 `send.js`

```js
#!/usr/bin/env node

const amqp = require('amqplib');

const queue = 'hello';
const text = 'Hello World!';

(async () => {
  let connection;
  try {
    connection = await amqp.connect('amqp://admin:express@localhost');
    const channel = await connection.createChannel();

    await channel.assertQueue(queue, { durable: false });

    // NB: `sentToQueue` and `publish` both return a boolean
    // indicating whether it's OK to send again straight away, or
    // (when `false`) that you should wait for the event `'drain'`
    // to fire before writing again. We're just doing the one write,
    // so we'll ignore it.
    channel.sendToQueue(queue, Buffer.from(text));
    console.log(" [x] Sent '%s'", text);
    await channel.close();
  }
  catch (err) {
    console.warn(err);
  }
  finally {
    if (connection) await connection.close();
  };
})();  
```

运行 `node send.js`

`receive.js`

```js
#!/usr/bin/env node

const amqp = require('amqplib');

const queue = 'hello';

(async () => {
  try {
    const connection = await amqp.connect('amqp://admin:express@localhost');
    const channel = await connection.createChannel();

    process.once('SIGINT', async () => { 
      await channel.close();
      await connection.close();
    });

    await channel.assertQueue(queue, { durable: false });
    await channel.consume(queue, (message) => {
      console.log(" [x] Received '%s'", message.content.toString());
    }, { noAck: true });

    console.log(' [*] Waiting for messages. To exit press CTRL+C');
  } catch (err) {
    console.warn(err);
  }
})();
```

运行 `node receive.js`

方法

| 方法                                         | 说明           |
| ------------------------------------------ | ------------ |
| `const connection = amqp.connect`          | 连接到 RabbitMQ |
| `connection.close`                         | 关闭连接         |
| `const channel = connection.createChannel` | 创建通道         |
| `channel.assertQueue`                      | 创建队列         |
| `channel.sendToQueue`                      | 发送消息         |
| `channel.consume`                          | 接收消息         |
| `channel.ack`                              | 确认消息         |
| `channel.ack`                              | 拒绝消息         |

参数

| 参数                 | 说明                                             |
| ------------------ | ---------------------------------------------- |
| `durable: true`    | 队列持久化，队列消息将会被写入磁盘。创建队列后这个参数无法修改，可以先删除之前的队列再修改。 |
| `persistent: true` | 消息持久化                                          |
| `noAck: true`      | 自动确认消息，只要收到了就会自动确认。如果设置成 `false` 就需要手动确认       |

## 直接发送邮件不等待

```js
sendMail(user.email, '注册成功', html)

// 由于改成异步的了，try-catch 捕获不到，可以在这里加上捕获
const sendMail = async (email, subject, html) => {
  try {
    await transporter.sendMail({
      from: process.env.MAILER_USER,
      to: email,
      subject,
      html
    })
  } catch (error) {
    console.log('邮件发送失败', error)
  }
}
```

## 在消息队列中发送邮件

好处：

- 解耦
- 增加并发
- 错误处理

`utils/rabbit-mq.js`

```js
const amqp = require('amqplib')
const sendMail = require('./mail')

let connection = null
let channel = null

/**
 * 连接 RabbitMQ 
 */
const connectToRabbitMQ = async () => {
  if (connection && channel) return

  try {
    connection = await amqp.connect(process.env.RABBITMQ_URL)
    channel = await connection.createChannel()
    await channel.assertQueue('mail_queue', { durable: true })
  } catch (error) {
    console.error('Failed to connect to RabbitMQ:', error)
  }
}

/**
 * 邮件队列生产者 发送消息
 * @param {*} msg 
 */
const mailProducer = async (msg) => {
  try {
    await connectToRabbitMQ()

    channel.sendToQueue('mail_queue', Buffer.from(JSON.stringify(msg)), {
      persistent: true
    })
  } catch (error) {
    console.log('Failed to send message to RabbitMQ:', error)
  }
}

/**
 * 邮件队列消费者 消费消息
 */
const mailConsumer = async () => {
  try {
    await connectToRabbitMQ()
    channel.consume('mail_queue', async (msg) => {
      const message = JSON.parse(msg.content.toString())
      await sendMail(message.to, message.subject, message.html)
    }, { noAck: true })
  } catch (error) {
    console.log('Failed to consume message from RabbitMQ:', error)
  }
}

module.exports = {
  mailProducer,
  mailConsumer
}
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
  try {
    await transporter.sendMail({
      from: process.env.MAILER_USER,
      to: email,
      subject,
      html
    })
  } catch (error) {
    console.log('邮件发送失败', error)
  }
}

module.exports = sendMail
```

`routes/auth.js` 中注册完成之后使用生产者

```js
const user = await User.create(body)
delete user.dataValues.password  // 不把密码返回给客户端

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
const msg = {
	to: user.email,
	subject: '注册成功',
	html
}
await mailProducer(msg)

success(res, '用户注册成功', { user }, 201)
```

`app.js` 中消费者，只要检测到任务完成直接消费掉

```js
const { mailConsumer } = require('./utils/rabbit-mq')
!(async () => {
  await mailConsumer()
  console.log('RabbitMQ 队列消费者启动成功')
})()
```

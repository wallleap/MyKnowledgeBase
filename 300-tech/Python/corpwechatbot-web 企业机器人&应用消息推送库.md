---
title: corpwechatbot-web 企业机器人&应用消息推送库
date: 2024-10-21T12:04:30+08:00
updated: 2024-10-21T12:18:03+08:00
dg-publish: false
source: //gist.github.com/GentleCP/5d02f4e84b8c8905bcf67643223cd499
---

[corpwechatbot： 企业微信消息推送助手 (gentlecp.com)](https://corpwechatbot.gentlecp.com/)

[GentleCP/corpwechatbot: 企业微信的python封装接口，简易上手，开箱即用，一行代码实现消息推送。A more convenient python wrapper interface of corpwechat(wework, wecom) official API, use only one line of code to send your messages to your corpwechat(wework, wecom) . (github.com)](https://github.com/GentleCP/corpwechatbot)

## 使用

```sh
$ pip install -U corpwechatbot
```

## 应用消息推送

### 推送接口演示

该推送会直接传至你的个人微信上，你会像收到好友消息一样收到通知信息，你需要先初始化一个 `AppMsgSender` 实例对象，如下：

```python
import cptools
from corpwechatbot.app import AppMsgSender
# from corpwehcatbot import AppMsgSender  # both will work

app = AppMsgSender(corpid='',  # 你的企业id
                   corpsecret='',  # 你的应用凭证密钥
                   agentid='', # 你的应用id
                   log_level=cptools.INFO, # 设置日志发送等级，INFO, ERROR, WARNING, CRITICAL,可选
                   proxies={'http':'http:example.com', 'https':'https:example.com'}  # 设置代理，可选
                   )   

# 如果你在本地配置添加了企业微信本地配置文件，也可以直接初始化AppMsgSender，而无需再显式传入密钥参数
# app = AppMsgSender()
```

> 在 `v0.3.0` 之后，你可以创建多个 `AppMsgSender`，以实现通过多个不同的应用的消息发送，这让你可以实现在一个项目中跨用户、跨企业的消息通知，下面是一个例子
>
> `app1 = AppMsgSender(corpid='1',  # 你的企业id                    corpsecret='1',  # 你的应用凭证密钥                    agentid='1')   # 你的应用id app2 = AppMsgSender(corpid='2',  # 你的企业id                    corpsecret='2',  # 你的应用凭证密钥                    agentid='2')   # 你的应用id app1.send_text('App1的消息')  app2.send_text('App2的消息')`

完成实例创建之后，你可以通过接口实现需要的信息推送，具体包括：

```python
app.send_text(content="如果我是DJ，你会爱我吗？")
app.send_image(image_path='test.png') # 图片存储路径
app.send_voice(voice_path='test.amr')
app.send_video(video_path='test.mp4')
app.send_file(file_path='test.txt')
app.send_markdown(content='# 面对困难的秘诀 \n > 加油，奥利给！')
app.send_news(title='性感刘公，在线征婚',
              desp='刘公居然要征婚了？这到底是人性的扭曲，还是道德的沦丧？...',
              url='https://blog.gentlecp.com',
              picurl='https://gitee.com/gentlecp/ImgUrl/raw/master/20210313141425.jpg')

app.send_mpnews(title='你好，我是CP',
               image_path='data/test.png',
               content='<a href="https://blog.gentlecp.com">Hello World</a>',
               content_source_url='https://blog.gentlecp.com',
               author='GentleCP',
               digest='这是一段描述',
               safe=1)

app.send_card(title='真骚哥出柜',
              desp='真骚哥竟然出柜了？对象竟然是他...',
              url='https://blog.gentlecp.com',
              btntxt='一睹为快')

btn = [{
  "key": "yes",
  "name": "好的",
  "color":"red",
  "is_bold": True,
},
  {
    "key": "no",
    "name": "wdnmd"
  }
]
app.send_taskcard(title="老板的消息",
                  desp="下个月工资减半",
                  url="http://127.0.0.1",
                  btn=btn,
                  task_id='12323',) # task_id在应用中是唯一的

```

所有应用推送消息有几个共同参数，用于指定发送消息的特性，如下

- `touser`: 要发送的用户，通过列表划分，输入成员 ID，默认发送给全体
- `toparty`: 要发送的部门，通过列表划分，输入部门 ID，当 touser 为 `@all` 时忽略
- `totag`: 发送给包含指定标签的人，通过列表划分，输入标签 ID，当 touser 为 `@all` 时忽略
- `safe`(该参数并非所有接口都支持，使用时请确认): 是否是保密消息，`0` 表示可对外分享，`1` 表示不能分享且内容显示水印，默认为 `0`

```python
# 一个演示程序
from corpwechatbot.app import AppMsgSender

app = AppMsgSender(corpid='',  # 你的企业id
                   corpsecret='',  # 你的应用凭证密钥
                   agentid='')   # 你的应用id

app.send_text(content='Hello',
              touser=['sb'],
              toparty=['1'],
              totag=['1'],
              safe=1)
```

⚠️注意！在指定 `toparty` 参数的时候，请确保你应用的可见范围包括该部门，否则会发送失败！很多失败情况都是应用的可见范围没设置正确导致！！

![](https://cdn.wallleap.cn/img/pic/illustration/202410211213819.png?imageSlim)

**获取相关用户数据的方法**

进入企业微信后台 ->通讯录

- 如何获取 userID
		![](https://cdn.wallleap.cn/img/pic/illustration/202410211213723.png?imageSlim)
- 如何获取 partyID
		![](https://cdn.wallleap.cn/img/pic/illustration/202410211214799.png?imageSlim)
- 如何获取 tagID
		![](https://cdn.wallleap.cn/img/pic/illustration/202410211214197.png?imageSlim)

## 应用群聊消息推送

应用群聊消息推送是基于应用（app）创建的群聊实现应用在群聊中的消息推送，目前，其推送的消息只能在**企业微信**中查看，不会在**个人微信**中展示。应用群聊消息推送的好处有几点：

1. 最少 2 个人即可建立群聊，企业微信（微信）默认都需要三人以上才能建立群聊，建立群聊后还可以添加群聊机器人（webhook）
2. 应用推送的消息在群聊中展示，而不是单个应用交互的方式，每个人与应用的交互在群聊中展示（大家都看得到）
3. 相比群聊机器人更丰富的消息推送形式（支持除任务卡片和小程序消息以外的所有应用消息推送）

### 推送接口演示 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/appchat_msg_send/#_2 "Permanent link")

如果你已经掌握了**应用消息推送**，那么你只需要在原来推送消息的基础上添加一个参数 `chatid`(群聊 id) 即可发送对应消息到群聊，在推送消息之前，需要先创建一个群聊。

> ⚠️请先确保加入群聊的用户都在应用的可见范围之内！！！

### 群聊创建 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/appchat_msg_send/#_3 "Permanent link")

`from corpwechatbot import AppMsgSender  app = AppMsgSender() res = app.create_chat(users=['zhangsan', 'lisi'], owner='zhangsan', name="test", chatid="123", show_chat=True) print(res)`

> ⚠️注意此时在企业微信中还是看不到群聊的，但群聊已经创建了，需要发送一条消息，让群聊显示出来，请务必保存好返回得到的 `chatid`，这是后面消息推送的主要参数
>
> 😏 `v0.6.2` 以后在创建群聊的时候可以指定 `show_chat` 参数（默认为 True），表示是否在群聊创建后发送一条消息到创建的群聊，让其在会话列表中显示出来，消息会包含群聊的 id 信息

### 消息推送 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/appchat_msg_send/#_4 "Permanent link")

`from corpwechatbot import AppMsgSender  app = AppMsgSender() app.send_text("hello world", chatid="123")`

将应用消息发送到群聊中非常简便，只需要在原来应用消息推送的基础上，添加一个 `chatid` 即可。 

其他消息的推送同理，需要注意的一点是更多的可传入参数 `touser`,`toparty`,`totag` 在群聊中不可用，`safe` 参数可用，更多内容和支持的消息类型参考 [官方文档](https://work.weixin.qq.com/api/doc/90000/90135/90248)

![](https://cdn.wallleap.cn/img/pic/illustration/202410211215796.png?imageSlim)

## 群聊机器人消息推送

### 推送接口演示 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#_2 "Permanent link")

> ⚠️注意！机器人发送的群聊只能在企业微信群聊中收到（个人微信中不会显示）！！！

要使用群聊机器人实现消息推送，你需要先实例化一个 `CorpWechatBot` 对象，如下：

`import cptools from corpwechatbot.chatbot import CorpWechatBot # fromo corpwechatbot import CorpWechatBot  # both will work bot = CorpWechatBot(key='',# 你的机器人key，通过群聊添加机器人获取                     log_level=cptools.INFO, # 设置日志发送等级，INFO, ERROR, WARNING, CRITICAL,可选                     proxies={'http':'http:example.com', 'https':'https:example.com'}  # 设置代理，可选                     )    # 如果你在本地配置添加了企业微信本地配置文件，也可以直接初始化CorpWechatBot，而无需再显式传入密钥参数 # bot = CorpWechatBot()`

接着，你就可以利用接口实现快速的机器人群里消息推送，具体包括：

### 文本消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#_3 "Permanent link")

`bot.send_text(content='Hello World')`

> ![](https://corpwechatbot.gentlecp.com/img/bot.png)

### 图片消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#_4 "Permanent link")

`bot.send_image(image_path='test.png')`

> ![img_8.png](https://corpwechatbot.gentlecp.com/img/bot_image.png)

### markdown 消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#markdown "Permanent link")

markdown 类型消息，支持 markdown 语法，**目前仅支持企业微信查看**，若输入内容以 `.md` 结尾，则会尝试读取对应文件内容发送。

`bot.send_markdown(content='# 面对困难的秘诀 \n > 加油，奥利给！')`

> ![img_9.png](https://corpwechatbot.gentlecp.com/img/bot_markdown.png)

### 图文消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#_5 "Permanent link")

`bot.send_news(title='性感刘公，在线征婚',               desp='刘公居然要征婚了？这到底是人性的扭曲，还是道德的沦丧？...',               url='https://blog.gentlecp.com',               picurl='https://gitee.com/gentlecp/ImgUrl/raw/master/20210313141425.jpg')`

> ![img_10.png](https://corpwechatbot.gentlecp.com/img/bot_news.png)

### 更多参数使用 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#_6 "Permanent link")

上面只是简单地列出了每个消息推送接口的使用，对于一般使用已经足够了，如果你还有更细致的要求，例如@某人或@全体成员，需要配置以下参数：

群聊消息中发送 text 消息时可以指定 `@` 的成员

- `mentioned_list:[]`: userid 列表，提醒群众某个成员，userid 通过企业通讯录查看，`@all` 则提醒所有人
- `mentioned_mobile_list:[]`: 手机号列表，提醒手机号对应的成员，`@all` 代表所有人，当不清楚 userid 时可替换

`# 一个演示程序 from corpwechatbot.chatbot import CorpWechatBot  bot = CorpWechatBot(key='')  # 你的机器人key，通过群聊添加机器人获取 bot.send_text(content='Hello World',               mentioned_list=['sb'],               mentioned_mobile_list=['110'])`

> ![img_11.png](https://corpwechatbot.gentlecp.com/img/bot_at.png)

### 获取相关用户数据方法 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/bot_msg_send/#_7 "Permanent link")

进入企业微信后台 ->通讯录

- 如何获取 userID
		![](https://corpwechatbot.gentlecp.com/img/get_userid.png)
- 如何获取 mobile

## 回调配置

### 配置步骤 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/callback_configuration/#_2 "Permanent link")

> 本部分主要讲解如何配置企业微信的回调机制，主要参考 [官方回调配置](https://open.work.weixin.qq.com/api/doc/90000/90135/90930) 和 [官方接口代码](https://github.com/sbzhu/weworkapi_python)

目标：添加回调配置，支持更丰富的应用场景，如

- 用户发送消息给应用，识别关键词，返回不同的消息内容
- 用户点击应用菜单，转换为相应指令，执行自动化任务，接收任务卡片消息，根据卡片内容，用户可选择在移动端点击反馈
- 服务端（你的服务器）与移动端（你的微信&企业微信）实现更 amazing 的交互模式（💯如聊天机器人）

要求

- 一台有独立 ip 的服务器
- 简单的 `fastapi` 用法
- 在企业微信后台进入应用管理，选择一个应用，找到接收消息 ![](https://cdn.jsdelivr.net/gh/GentleCP/ImgUrl/20210626140957.png)
- 设置 API 接收消息的三个选项，如下 ![](https://cdn.jsdelivr.net/gh/GentleCP/ImgUrl/20210626141701.png)
- 参考 [corpwechatbot-web (github.com)](https://gist.github.com/GentleCP/5d02f4e84b8c8905bcf67643223cd499)，在服务器上运行 `web.py`

		`python3 web.py -p=8000 -t="token" -a="aeskey" -c="corpid"`
		
		确认你的web服务成功启动，并能通过公网访问

4.点击保存，如果正常，会提示 API 设置成功，记住这些配置信息 ![](https://cdn.jsdelivr.net/gh/GentleCP/ImgUrl/20210626143027.png)

至此，你的服务器已经具备基本的请求和响应功能了，由于 post 的请求根据用户的需求各异，目前只是完成了一个简单的发送和响应功能，后续有时间再完善。

下面是请求响应的例子： ![](https://corpwechatbot.gentlecp.com/img/callback_test.gif)

`ierror.py`

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
#########################################################################
# Author: jonyqin
# Created Time: Thu 11 Sep 2014 01:53:58 PM CST
# File Name: ierror.py
# Description:定义错误码含义 
#########################################################################
WXBizMsgCrypt_OK = 0
WXBizMsgCrypt_ValidateSignature_Error = -40001
WXBizMsgCrypt_ParseXml_Error = -40002
WXBizMsgCrypt_ComputeSignature_Error = -40003
WXBizMsgCrypt_IllegalAesKey = -40004
WXBizMsgCrypt_ValidateCorpid_Error = -40005
WXBizMsgCrypt_EncryptAES_Error = -40006
WXBizMsgCrypt_DecryptAES_Error = -40007
WXBizMsgCrypt_IllegalBuffer = -40008
WXBizMsgCrypt_EncodeBase64_Error = -40009
WXBizMsgCrypt_DecodeBase64_Error = -40010
WXBizMsgCrypt_GenReturnXml_Error = -40011
```

`web.py`

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-
"""
-----------------File Info-----------------------
Name: web.py
Description: web api support
Author: GentleCP
Email: me@gentlecp.com
Create Date: 2021/6/19
-----------------End-----------------------------
"""
import argparse
from fastapi import FastAPI
from fastapi import Response, Request
from WXBizMsgCrypt3 import WXBizMsgCrypt
from xml.etree.ElementTree import fromstring
import uvicorn

app = FastAPI()

def parse_args():
    arg_parser = argparse.ArgumentParser()
    arg_parser.add_argument('--port', '-p', default=8000, type=int, help="port to build web server")
    arg_parser.add_argument('--token', '-t', type=str, help='token set in corpwechat app')
    arg_parser.add_argument('--aeskey', '-a', type=str, help='encoding aeskey')
    arg_parser.add_argument('--corpid', '-c', type=str, help='your corpwechat id')
    args = arg_parser.parse_args()
    return args

args = parse_args()
wxcpt = WXBizMsgCrypt(args.token, args.aeskey, args.corpid)

@app.get("/")
async def verify(msg_signature: str,
                 timestamp: str,
                 nonce: str,
                 echostr: str):
    '''
    验证配置是否成功，处理get请求
    :param msg_signature:
    :param timestamp:
    :param nonce:
    :param echostr:
    :return:
    '''
    ret, sEchoStr = wxcpt.VerifyURL(msg_signature, timestamp, nonce, echostr)
    if ret == 0:
        return Response(content=sEchoStr.decode('utf-8'))
    else:
        print(sEchoStr)

@app.post("/")
async def recv(msg_signature: str,
               timestamp: str,
               nonce: str,
               request: Request):
    '''
    接收用户消息，可进行被动响应
    :param msg_signature:
    :param timestamp:
    :param nonce:
    :param request:
    :return:
    '''
    body = await request.body()
    ret, sMsg = wxcpt.DecryptMsg(body.decode('utf-8'), msg_signature, timestamp, nonce)
    decrypt_data = {}
    for node in list(fromstring(sMsg.decode('utf-8'))):
        decrypt_data[node.tag] = node.text
    # 解析后得到的decrypt_data: {"ToUserName":"企业号", "FromUserName":"发送者用户名", "CreateTime":"发送时间", "Content":"用户发送的内容", "MsgId":"唯一id，需要针对此id做出响应", "AagentID": "应用id"}
    # 用户应根据Content的内容自定义要做出的行为，包括响应返回数据，如下例子，如果发送的是123，就返回hello world

    # 处理任务卡片消息
    if decrypt_data.get('EventKey', '') == 'no':
        # 返回信息
        sRespData="""<xml>
   <ToUserName>{to_username}</ToUserName>
   <FromUserName>{from_username}</FromUserName>
   <CreateTime>{create_time}</CreateTime>
   <MsgType>update_taskcard</MsgType>
   <TaskCard>
       <ReplaceName>已处理</ReplaceName>
   </TaskCard>
</xml>
""".format(to_username=decrypt_data['ToUserName'],
           from_username=decrypt_data['FromUserName'],
           create_time=decrypt_data['CreateTime'],
           event_key=decrypt_data['EventKey'],
           agentid=decrypt_data['AgentId'])
    # 处理文本消息
    if decrypt_data.get('Content', '') == '我帅吗':
        sRespData = """<xml>
   <ToUserName>{to_username}</ToUserName>
   <FromUserName>{from_username}</FromUserName> 
   <CreateTime>{create_time}</CreateTime>
   <MsgType>text</MsgType>
   <Content>{content}</Content>
</xml>
""".format(to_username=decrypt_data['ToUserName'],
           from_username=decrypt_data['FromUserName'],
           create_time=decrypt_data['CreateTime'],
           content="帅得一逼",)
    ret, send_msg = wxcpt.EncryptMsg(sReplyMsg=sRespData, sNonce=nonce)
    if ret == 0:
        return Response(content=send_msg)
    else:
        print(send_msg)

if __name__ == "__main__":
    uvicorn.run("web:app", port=args.port, host='0.0.0.0', reload=False)
```

`WXBizMsgCrypt3.py`

```python
#!/usr/bin/env python
# -*- encoding:utf-8 -*-

""" 对企业微信发送给企业后台的消息加解密示例代码.
@copyright: Copyright (c) 1998-2014 Tencent Inc.
"""
# ------------------------------------------------------------------------
import logging
import base64
import random
import hashlib
import time
import struct
from Crypto.Cipher import AES
import xml.etree.cElementTree as ET
import socket

import ierror


"""
关于Crypto.Cipher模块，ImportError: No module named 'Crypto'解决方案
请到官方网站 https://www.dlitz.net/software/pycrypto/ 下载pycrypto。
下载后，按照README中的“Installation”小节的提示进行pycrypto安装。
"""


class FormatException(Exception):
    pass


def throw_exception(message, exception_class=FormatException):
    """my define raise exception function"""
    raise exception_class(message)


class SHA1:
    """计算企业微信的消息签名接口"""

    def getSHA1(self, token, timestamp, nonce, encrypt):
        """用SHA1算法生成安全签名
        @param token:  票据
        @param timestamp: 时间戳
        @param encrypt: 密文
        @param nonce: 随机字符串
        @return: 安全签名
        """
        try:
            sortlist = [token, timestamp, nonce, encrypt]
            sortlist.sort()
            sha = hashlib.sha1()
            sha.update("".join(sortlist).encode())
            return ierror.WXBizMsgCrypt_OK, sha.hexdigest()
        except Exception as e:
            logger = logging.getLogger()
            logger.error(e)
            return ierror.WXBizMsgCrypt_ComputeSignature_Error, None


class XMLParse:
    """提供提取消息格式中的密文及生成回复消息格式的接口"""

    # xml消息模板
    AES_TEXT_RESPONSE_TEMPLATE = """<xml>
<Encrypt><![CDATA[%(msg_encrypt)s]]></Encrypt>
<MsgSignature><![CDATA[%(msg_signaturet)s]]></MsgSignature>
<TimeStamp>%(timestamp)s</TimeStamp>
<Nonce><![CDATA[%(nonce)s]]></Nonce>
</xml>"""

    def extract(self, xmltext):
        """提取出xml数据包中的加密消息
        @param xmltext: 待提取的xml字符串
        @return: 提取出的加密消息字符串
        """
        try:
            xml_tree = ET.fromstring(xmltext)
            encrypt = xml_tree.find("Encrypt")
            return ierror.WXBizMsgCrypt_OK, encrypt.text
        except Exception as e:
            logger = logging.getLogger()
            logger.error(e)
            return ierror.WXBizMsgCrypt_ParseXml_Error, None

    def generate(self, encrypt, signature, timestamp, nonce):
        """生成xml消息
        @param encrypt: 加密后的消息密文
        @param signature: 安全签名
        @param timestamp: 时间戳
        @param nonce: 随机字符串
        @return: 生成的xml字符串
        """
        resp_dict = {
            'msg_encrypt': encrypt,
            'msg_signaturet': signature,
            'timestamp': timestamp,
            'nonce': nonce,
        }
        resp_xml = self.AES_TEXT_RESPONSE_TEMPLATE % resp_dict
        return resp_xml


class PKCS7Encoder():
    """提供基于PKCS7算法的加解密接口"""

    block_size = 32

    def encode(self, text):
        """ 对需要加密的明文进行填充补位
        @param text: 需要进行填充补位操作的明文
        @return: 补齐明文字符串
        """
        text_length = len(text)
        # 计算需要填充的位数
        amount_to_pad = self.block_size - (text_length % self.block_size)
        if amount_to_pad == 0:
            amount_to_pad = self.block_size
        # 获得补位所用的字符
        pad = chr(amount_to_pad)
        return text + (pad * amount_to_pad).encode()

    def decode(self, decrypted):
        """删除解密后明文的补位字符
        @param decrypted: 解密后的明文
        @return: 删除补位字符后的明文
        """
        pad = ord(decrypted[-1])
        if pad < 1 or pad > 32:
            pad = 0
        return decrypted[:-pad]


class Prpcrypt(object):
    """提供接收和推送给企业微信消息的加解密接口"""

    def __init__(self, key):

        # self.key = base64.b64decode(key+"=")
        self.key = key
        # 设置加解密模式为AES的CBC模式
        self.mode = AES.MODE_CBC

    def encrypt(self, text, receiveid):
        """对明文进行加密
        @param text: 需要加密的明文
        @return: 加密得到的字符串
        """
        # 16位随机字符串添加到明文开头
        text = text.encode()
        text = self.get_random_str() + struct.pack("I", socket.htonl(len(text))) + text + receiveid.encode()

        # 使用自定义的填充方式对明文进行补位填充
        pkcs7 = PKCS7Encoder()
        text = pkcs7.encode(text)
        # 加密
        cryptor = AES.new(self.key, self.mode, self.key[:16])
        try:
            ciphertext = cryptor.encrypt(text)
            # 使用BASE64对加密后的字符串进行编码
            return ierror.WXBizMsgCrypt_OK, base64.b64encode(ciphertext)
        except Exception as e:
            logger = logging.getLogger()
            logger.error(e)
            return ierror.WXBizMsgCrypt_EncryptAES_Error, None

    def decrypt(self, text, receiveid):
        """对解密后的明文进行补位删除
        @param text: 密文
        @return: 删除填充补位后的明文
        """
        try:
            cryptor = AES.new(self.key, self.mode, self.key[:16])
            # 使用BASE64对密文进行解码，然后AES-CBC解密
            plain_text = cryptor.decrypt(base64.b64decode(text))
        except Exception as e:
            logger = logging.getLogger()
            logger.error(e)
            return ierror.WXBizMsgCrypt_DecryptAES_Error, None
        try:
            pad = plain_text[-1]
            # 去掉补位字符串
            # pkcs7 = PKCS7Encoder()
            # plain_text = pkcs7.encode(plain_text)
            # 去除16位随机字符串
            content = plain_text[16:-pad]
            xml_len = socket.ntohl(struct.unpack("I", content[: 4])[0])
            xml_content = content[4: xml_len + 4]
            from_receiveid = content[xml_len + 4:]
        except Exception as e:
            logger = logging.getLogger()
            logger.error(e)
            return ierror.WXBizMsgCrypt_IllegalBuffer, None

        if from_receiveid.decode('utf8') != receiveid:
            return ierror.WXBizMsgCrypt_ValidateCorpid_Error, None
        return 0, xml_content

    def get_random_str(self):
        """ 随机生成16位字符串
        @return: 16位字符串
        """
        return str(random.randint(1000000000000000, 9999999999999999)).encode()


class WXBizMsgCrypt(object):
    # 构造函数
    def __init__(self, sToken, sEncodingAESKey, sReceiveId):
        try:
            self.key = base64.b64decode(sEncodingAESKey + "=")
            assert len(self.key) == 32
        except:
            throw_exception("[error]: EncodingAESKey unvalid !", FormatException)
            # return ierror.WXBizMsgCrypt_IllegalAesKey,None
        self.m_sToken = sToken
        self.m_sReceiveId = sReceiveId

        # 验证URL
        # @param sMsgSignature: 签名串，对应URL参数的msg_signature
        # @param sTimeStamp: 时间戳，对应URL参数的timestamp
        # @param sNonce: 随机串，对应URL参数的nonce
        # @param sEchoStr: 随机串，对应URL参数的echostr
        # @param sReplyEchoStr: 解密之后的echostr，当return返回0时有效
        # @return：成功0，失败返回对应的错误码

    def VerifyURL(self, sMsgSignature, sTimeStamp, sNonce, sEchoStr):
        sha1 = SHA1()
        ret, signature = sha1.getSHA1(self.m_sToken, sTimeStamp, sNonce, sEchoStr)
        if ret != 0:
            return ret, None
        if not signature == sMsgSignature:
            return ierror.WXBizMsgCrypt_ValidateSignature_Error, None
        pc = Prpcrypt(self.key)
        ret, sReplyEchoStr = pc.decrypt(sEchoStr, self.m_sReceiveId)
        return ret, sReplyEchoStr

    def EncryptMsg(self, sReplyMsg, sNonce, timestamp=None):
        # 将企业回复用户的消息加密打包
        # @param sReplyMsg: 企业号待回复用户的消息，xml格式的字符串
        # @param sTimeStamp: 时间戳，可以自己生成，也可以用URL参数的timestamp,如为None则自动用当前时间
        # @param sNonce: 随机串，可以自己生成，也可以用URL参数的nonce
        # sEncryptMsg: 加密后的可以直接回复用户的密文，包括msg_signature, timestamp, nonce, encrypt的xml格式的字符串,
        # return：成功0，sEncryptMsg,失败返回对应的错误码None
        pc = Prpcrypt(self.key)
        ret, encrypt = pc.encrypt(sReplyMsg, self.m_sReceiveId)
        encrypt = encrypt.decode('utf8')
        if ret != 0:
            return ret, None
        if timestamp is None:
            timestamp = str(int(time.time()))
        # 生成安全签名
        sha1 = SHA1()
        ret, signature = sha1.getSHA1(self.m_sToken, timestamp, sNonce, encrypt)
        if ret != 0:
            return ret, None
        xmlParse = XMLParse()
        return ret, xmlParse.generate(encrypt, signature, timestamp, sNonce)

    def DecryptMsg(self, sPostData, sMsgSignature, sTimeStamp, sNonce):
        # 检验消息的真实性，并且获取解密后的明文
        # @param sMsgSignature: 签名串，对应URL参数的msg_signature
        # @param sTimeStamp: 时间戳，对应URL参数的timestamp
        # @param sNonce: 随机串，对应URL参数的nonce
        # @param sPostData: 密文，对应POST请求的数据
        #  xml_content: 解密后的原文，当return返回0时有效
        # @return: 成功0，失败返回对应的错误码
        # 验证安全签名
        xmlParse = XMLParse()
        ret, encrypt = xmlParse.extract(sPostData)
        if ret != 0:
            return ret, None
        sha1 = SHA1()
        ret, signature = sha1.getSHA1(self.m_sToken, sTimeStamp, sNonce, encrypt)
        if ret != 0:
            return ret, None
        if not signature == sMsgSignature:
            return ierror.WXBizMsgCrypt_ValidateSignature_Error, None
        pc = Prpcrypt(self.key)
        ret, xml_content = pc.decrypt(encrypt, self.m_sReceiveId)
        return ret, xml_content
```

## 本地密钥配置与终端消息推送

### 企业微信本地配置文件 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/terminal_msg_send/#_2 "Permanent link")

默认情况下，在初始化推送实例的时候，都会要求传入相应的关键参数，但在程序中直接写这些敏感信息并不是一个好选项，因此新版本支持在本地用户目录 (`~`) 下创建一个 `.corpwechatbot_key` 文件，写入如下配置信息：

`[app] corpid=   corpsecret= agentid= [chatbot] key=`

这样，在实例化一个类时，就不需要再显式地传入 `corpid,corpsecret,key` 等参数了，如下：

``from corpwechatbot.app import AppMsgSender from corpwechatbot import CorpWechatBot  app = AppMsgSender()  # 不传参，直接从本地`~/.corpwechatbot_key`读取 app.send_text(content="如果我是DJ，你会爱我吗？")  bot = CorpWechatBot() bot.send_text(content='Hello World!')``

同时由于配置信息都统一存储在本地，意味着你任何一个新的项目也可以不需要专门设置一个文件来存储这些敏感数据了

> 如果你不知道 `~` 目录在哪，可以通过如下代码获取：
>
> `from pathlib import Path  print(Path.home())`
>
> 在 `v0.6.0` 之后，支持本地创建多个密钥文件，并在实例化 `AppMsgSender` 的时候传入文件路径进行指定密钥的实例化，如下：
>
> `from corpwechatbot.app import AppMsgSender  app = AppMsgSender(key_path="~/.corpwechatbot_key2")`

### 终端快速发送消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/terminal_msg_send/#_3 "Permanent link")

很多时候，我们希望直接在终端敲一行代码就将消息发送到了我们的手机上，**在 `v0.2.0` 版本后支持啦！！！** 使用前需先**配置本地密钥文件**

目前支持：

### 文本消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/terminal_msg_send/#_4 "Permanent link")

`# 应用消息 cwb -u='app' -t='hello world'  # 通过应用发送消息 cwb -t='hello world'  # 更快捷的方式  # 群聊机器人消息 cwb -u='bot' -t='hello world'`

### markdwn 消息 [¶](https://corpwechatbot.gentlecp.com/%E4%BD%BF%E7%94%A8%E6%95%99%E7%A8%8B/terminal_msg_send/#markdwn "Permanent link")

`cwb -u='app' -m='hello world'   cwb -m='# Hello World'  # 更快捷的方式  # 群聊机器人消息 cwb -u='bot' -m='hello world'`

- **动图演示**

![](https://corpwechatbot.gentlecp.com/img/teminal_msgsend.gif)

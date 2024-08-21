---
title: 如何基于 ActionCable 搭建 webrtc 的服务
date: 2024-08-16T09:47:00+08:00
updated: 2024-08-21T10:32:32+08:00
dg-publish: false
---

如果你对 webrtc 感兴趣，但是对于如何基于 rails 实现这个功能有点无从下手或者还有些迷惑的话，这篇教程也许会给予一定的帮助

- 代码地址：[https://github.com/BranLiang/demo-rails-webrtc](https://github.com/BranLiang/demo-rails-webrtc)
- demo 体验链接：[https://demo-rails-webrtc.herokuapp.com/](https://demo-rails-webrtc.herokuapp.com/) （也支持 iphone 的 safari 和 android 浏览器）

## 实现原理

[https://medium.com/@BranLiang/a-complete-guide-to-webrtc-with-ruby-on-rails-9ea68e67154e](https://medium.com/@BranLiang/a-complete-guide-to-webrtc-with-ruby-on-rails-9ea68e67154e)

具体的实现原理我在上面的链接有详细的描述，这里简单说明一下背后的原理。既然使用了 webrtc 的接口，背后肯定是遵循的 webrtc 的规则，首先需要一个 signaling server，这相当于一个被所有人认可的一个中转机构，接收一个人的信息并传递给另外一个人，在这个 demo 项目里面，充当这个角色的就是 ActionCable，因为 ActionCable 能够比较方便的做到主动往客户端推送消息，当然你也可以选择其他的消息订阅服务所谓你的 signaling server，本身 webrtc 这个技术对于 signaling sever 没有做任何规范与限制。其次你需要一个客户端来与 signaling server 进行交互，在网页这个领域，应该除了 js 没有别的选择了，这里选择了 stimulus 框架来进行这个交互操作。有了上述元素之后就需要按照 webrtc 的流程一步步走下去，直到两台设备之间能够直接进行 P2P 的沟通。下面，就让我们一步步走下去，看看到底是如何建立一个简单的 webrtc 通信的。

## 第一步：创建 RTCPeerConnection

使用 webrtc 通信的第一步是处理话一个连接，每台设备都至少需要一个这么连接，这个连接会存储 `LocalDescription` 和 `RemoteDescription` 分别表示本地的配置以及连接的另一方的配置信息，同时还会存储对方连接的信息，一般称之为 `iceCandidates`。

```
new RTCPeerConnection({
   iceServers: servers // STUN/TURN server的配置信息，demo中来自twilio提供的服务，属于必须配置的参数
});
```

## 第二步：获取自身的配置信息并发送给对方

初始化连接之后，你需要获取自身的配置，并通过 signaling server 传递到另外一个客户端，获取的方式是多种多样的，下面是其中一种，通过 `createOffer` 这个接口获取本地的配置，示例代码如下：

```
this.peerConnection.createOffer(
  function(offer) {
    that.peerConnection.setLocalDescription(offer)
    that.channel.send({
      type: "OFFER",
      name: that.identifier,
      sdp: JSON.stringify(offer)
    })
  },
  function(err) {
    console.log(err)
  }
)
```

action cable 方面处理的逻辑非常简单，基本上除了 token，其余的请求都是传过来什么，就播送出去什么，最理想的状态是可以指定接收的订阅者，但是通过 action cable 并不能实现这一点，着实有些遗憾。

```
def receive(data)
  case data["type"]
  when "OFFER"
    ActionCable.server.broadcast(room_name, data)
  end
end
```

## 第三步：对方收到 offer 请求，并回复

通过上面的 action cable 数据播送，远在其他地方的客户端会接收到相应的请求消息，这时候它会保存对应的数据，并和请求方一样生成自身的 description，并通过 signaling server 传递出来，此时作为 offer 的接收方，已经获取了两边的配置数据。

```
createAnswer(offer) {
  this.connected = true
  let rtcOffer = new RTCSessionDescription(offer);
  let that = this
  this.peerConnection.setRemoteDescription(rtcOffer);
  this.peerConnection.createAnswer(
    function(answer) {
      that.peerConnection.setLocalDescription(answer)
      that.channel.send({
        type: "ANSWER",
        name: that.identifier,
        sdp: JSON.stringify(answer)
      })
    },
    function(err) {
      console.log(err)
    }
  )
}
```

## 第三步：请求方获取配置信息

上面一步中，对方回复了配置消息，那么很自然的请求方就会处理这个消息并记录下来，到这一步结束，两个客户端都保存了自身以及对方的配置信息。

```
receiveAnswer(answer) {
  let rtcAnswer = new RTCSessionDescription(answer);
  this.peerConnection.setRemoteDescription(rtcAnswer)
}
```

## 第四步：加载对方的连接方式

理想的情况下，两边应该是可以正常开始通信了，但是如果你测试的话，发现并不能，因为虽然知道了对方的配置，但是如何访问对方仍然是未知的，这个需要通过第一步配置 servers 来来获取自身的连接信息（一般是 ip 等信息），然后通过 signaling server 传递给对方，这样对方就能够顺利的直接访问自己了！

```
  this.peerConnection.onicecandidate = ({candidate}) => {
    this.channel.send({
      type: "CANDIDATE",
      name: this.identifier,
      sdp: JSON.stringify(candidate)
    })
  }

addCandidate(candidate) {
  let rtcCandidate = new RTCIceCandidate(candidate);
  this.peerConnection.addIceCandidate(rtcCandidate)
}
```

## 完成连接

到这一步之后，两个客户端之间的连接就已经完成了。需要如何利用这个连接就全凭大家的发挥了！欢迎留言讨论

---

挺好的，但是在非 lan 网络下，应该还需要类型 stun/turn 之类的服务来做打洞或中继。

---

bigbluebutton 挺好的呀，配套的前端 greenlight 也是基于 Rails 开发

---

Agora.io

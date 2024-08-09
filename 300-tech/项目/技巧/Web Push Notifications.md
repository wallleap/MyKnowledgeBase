---
title: "Web Push Notifications: 保存推送的subscription信息"
date: 2024-08-08 22:59
updated: 2024-08-08 23:00
---

## Web Push Notifications: 介绍，环境搭建和 server keys 的使用

这里的内容是由像和你一样的工程师们一行一行创造出来的，为了保护你的权益，同时也为了激励我们继续创造更好的内容，蛋人网所有内容禁止一切形式的转载和复制

Push Notification（通知推送）在手机端大家都已经非常熟悉了，而 Web 端基于网页的通知推送现在也慢慢被很多网站应用，尤其是技术类和技术类的服务网站，这个还是因为 HTML5 和 JavaScript 的发展太过迅猛，尤其是 JavaScript，这个系列文档中我们来研究和实现一下 Web 端的通知推送。

### 准备

#### 目标

1. Web Push Notifications 前端实现
2. 后端推送实现

#### 学习要求

- HTML, JavaScript
- Ruby on Rails

Web 端的通知推送核心还是在于前端的 JavaScript，这个文档中后端是用 Ruby on Rails 来实现的，你可以根据自己的情况 (Node.js / Python / PHP)，用什么后端无所谓，后端是作为扩展讲述的。

> Web Push Notifications 要求 localhost 环境是可以使用 HTTP 的，**非 localhost 必须使用 HTTPS**，在浏览器中测试时这里一定要注意

#### 浏览器支持

Web 端推送得益于 HTML 的标准，接口还是比较统一的，不同的浏览器的实现基本是一样的，这也是相对于 APP 端推送的优势；但是我们知道不同浏览器支持的程度是有区别的，目前来说 Chrome 和 Firefox 支持的最完善的，IE 和 Edge 还不支持，Safari 是使用的苹果自己的 [机制](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/NotificationProgrammingGuideForWebsites/PushNotifications/PushNotifications.html#//apple_ref/doc/uid/TP40013225-CH3-SW1) 来实现的，**因此在这个文档中的代码是完全支持 Chrome 和 Firefox 的，IE/Edge/Safari 不支持**。

#### Web Push Notifications 使用到的技术

涉及到的知识点有两个，当然都是浏览器端的:

**Service Worker**：简单说就是在浏览器后台运行的一段 JavaScript 代码，即使用户没有打开你的网站。当从服务器端 push 消息时，Service Worker 会接收到。

**Notification**: 当你在 Service Worker 中收到推送消息时就可以调用 Notification 相关的 API 来展示通知并处理用户的点击事件，比如打开你的网站。

#### 数据交互流程

##### 用户接收推送

##### 服务器端发送推送

#### 科学上网

因为不同浏览器采用的推送中心的服务器是不一样的，Chrome 是 google 的服务器，在向用户发送推送消息时需要连接 google 的 API，所以这里大家应该都明白，在测试推送时科学上网是必须的，Firefox 的推送 API 暂时没有问题。

### 创建项目

上面说了，我们这里采用的是 Ruby on Rails 来开发的，你可以根据自己的爱好选择不同的后端语言和框架，原理是一样的。

这里采用的是 Rails 5.1 的版本，请保证你的环境满足需求:

```
# rails -v
Rails 5.1.1
```

**创建项目**

```
$ rails new --skip-test --skip-puma --skip-coffee --skip-turbolinks --skip-bundle web_push_sample
```

ruby gems 的安装源国内被墙了，因此上面跳过了自动运行 bundle，创建完项目之后，进入到项目目录中，编辑文件 Gemfile，修改第一行的 gem 源地址：

```
source 'https://rubygems.org'
```

改为:

```
 source 'https://gems.ruby-china.org'
```

然后安装 gem:

```
 $ bundle
```

为了简化测试，数据库这里采用了默认的 SQLite，如果你想用其他数据库，在上面的 `rails new` 命令中添加 `-d mysql` 参数就行了。

启动:

```
$ rails s
```

上面的命令会打开你的 localhost 的**3000**号端口，然后打开浏览器访问**localhost:3000**，如果能打开网页就证明项目已经可以了。

### 生成 server keys

这里的 server keys 简单来说就是验证开发者服务器和推送中心服务器的用户名和密码，和我们平时常见的其他 API 服务的 keys 没有什么区别。

**Web Push Protocol**

Web Push Notifications 的实现也是遵循的专用的 [Web Push Protocol](https://tools.ietf.org/html/draft-ietf-webpush-protocol) 协议，规定了推送的数据传递方式，这里最关键的就是 server keys 了，在上面的服务器端推送流程中我们知道，实际上是一个三方的通信过程，`浏览器`, `推送中心消息服务器` 和 `开发者服务器`，推送中心服务器就是不同浏览器自己的中心服务器，负责把由开发者服务器发出的推送消息推送到用户的浏览器中，这个过程我们不用关心。

**webpush gem**

生成 server keys 的方式由很多中，根据你现在不同的开发环境可以选择对应的第三方工具，我们这里用的 Ruby，因此可以使用和 [webpush](https://github.com/zaru/webpush) 这个 gem，编辑 Gemfile 这个文件，添加下面的内容:

```
gem 'webpush'
```

然后再次运行 `bundle` 命令。

**生成 server keys**

进入 `rails console`:

```
$ rails c
Loading development environment (Rails 5.1.1)
2.4.0 :001 > a = Webpush.generate_key
=> #<Webpush::VapidKey:3ff734519c24 :public_key=BPlmwZjgJ5g0SYNmVOtC5iYQSZxxQJPzinYC1KBnVCjL26w_B8bZDTSCmkuumAjlzK91merXNs_6zDlZKNJ8Na8= :private_key=Pb_a0VHwfCMDibzf69_FeNPuCCkoL5Q_efJQFw0o7t0=>
2.4.0 :003 > a.public_key
=> "BPlmwZjgJ5g0SYNmVOtC5iYQSZxxQJPzinYC1KBnVCjL26w_B8bZDTSCmkuumAjlzK91merXNs_6zDlZKNJ8Na8="
2.4.0 :004 > a.private_key
=> "Pb_a0VHwfCMDibzf69_FeNPuCCkoL5Q_efJQFw0o7t0="
```

手工复制下上面打印出的**public_key**和**private_key**，保存在你的 `config/application.rb` 文件中:

```
WEB_PUSH_PUBLIC_KEY = 'BOlN-gCdCk7NxYEQAV6xELMj0F1RrVBB0dx3PwXTdwkmnzjvZr2BS03W6p7Oq7b5I3e0i8ybNPEdYA4pBjJwqE8='
WEB_PUSH_PRIVATE_KEY = 'TQO4dfgILSsFuNZ3oH55O2Wk5dYPQzAIPdbz-h8NlLs='

module WebPushSample
  class Application < Rails::Application
    # ...
```

> 不要忘了把你的 rails 进程重启一下。

如果你在使用 nodejs，可以使用 `web-push` 这个 package。

### 使用 server keys

我们需要在前端用户接收推送时使用**public key**，因此这里需要来做一些工作，让我们能够在 JavaScript 读取在 `config/application.rb` 中配置的 public_key。

把 `app/assets/javascripts` 中的 `application.js` 重命名为 `application.js.erb`，这样就能够在里面写 erb 的代码了，然后在 `application.js.erb` 中放入以下内容：

```
window.AppConfig = {
  PUBLIC_SERVER_KEY: '<%= WEB_PUSH_PUBLIC_KEY %>',
}
```

上面的代码创建了一个全局的**AppConfig**的变量，把 Rails 中所配置的变量设置到 AppConfig 中，这样就能在前端使用**public key**了。

**然后再建立前端对应的测试页面**：

首先修改路由，打开 `config/routes.rb` 文件，修改为以下内容：

```
Rails.application.routes.draw do
  root 'welcome#index'
end
```

然后创建首页的 Controller，在 `app/controllers` 目录中建立文件 `welcome_controller.rb`:

```
class WelcomeController < ApplicationController
end
```

最后在 `app/views` 目录中建立目录 `welcome`，并在下面建立文件 `index.html.erb` 文件:

```
<h1>Web Push Notification</h1>
```

最后就可以测试下能否在前端页面中访问到**AppConfig**了，在浏览器中打开**localhost:3000**，然后在开发者工具 (比如你使用的是 Chrome) 中打开**Console**的 tab，运行下面的 JavaScript 代码：

```
> AppConfig
Object {PUBLIC_SERVER_KEY: "BOlN-gCdCk7NxYEQAV6xELMj0F1RrVBB0dx3PwXTdwkmnzjvZr2BS03W6p7Oq7b5I3e0i8ybNPEdYA4pBjJwqE8="}
```

这时你应该能看到上面的输出结果，这就表示配置工作已经做完了，接下来就可以正式开发 Push Notifications 功能了。

点击这里继续观看第二部分：[Web Push Notifications: 通知推送订阅功能开发](https://eggman.tv/lessons/web-push-notifications-subscription-development)

## Web Push Notifications: 通知推送订阅功能开发

这里的内容是由像和你一样的工程师们一行一行创造出来的，为了保护你的权益，同时也为了激励我们继续创造更好的内容，蛋人网所有内容禁止一切形式的转载和复制

> 这篇文章是 [**Web Push Notifications**](https://eggman.tv/c/s-web-push-notifications) 系列文章的第二部分，如果要观看第一部分请点击这里: [Web Push Notifications: 介绍，环境搭建和server keys的使用](https://eggman.tv/lessons/web-push-notifications-introduction-and-environment-setting-up)

在上一章节中我们已经准备好了项目，生成了**server keys**，以及在前端页面的 JavaScript 中也能访问到**public key**了，接下来就来开发推送通知的订阅功能，这部分主要集中在前端 JavaScript 的实现。

### 推送订阅实现

首先在 `app/assets/javascripts` 中建立一个 JS 文件 `push.js`，大部分逻辑都将在这个文件中实现。

打开 `push.js` 文件添加以下代码：

```
window.addEventListener('load', function() {
  if ('PushManager' in window) {
    askPermission()
    .then(function() {
      if ('serviceWorker' in navigator) {
        // 这个文件必须和当前网站在同一个域名下
        registerServiceWorker('/service-worker.js');
        navigator.serviceWorker.ready.then(function(serviceWorkerRegistration) {
          subscribePush(serviceWorkerRegistration);
        });
      } else {
        console.log("不支持Service Worker");
      }
    })
    .catch(function(err) {
      console.log('Notification被拒绝');
    })
  } else {
    console.log("不支持Push");
  }
})
```

我们监听了页面的 `load` 事件，然后：

- 首先判断当前浏览器是否支持 `PushManager` 也就是推送功能
- askPermission: 如果支持，调用 `askPermission` 方法来弹出通知授权的窗口，让用户来选择时候要接收推送通知
- 再次判断浏览器是否支持 `serviceWorker`
- registerServiceWorker: 调用 `registerServiceWorker` 方法注册 Service Worker
- navigator.serviceWorker.ready.then: 当 Service Worker 注册成功后
- subscribePush: 调用 `subscribePush` 方法来获取当前的浏览器的推送 subscription 信息，并发送到开发者服务器端保存起来，以便之后发送推送消息时使用

以上就是整个通知推送订阅的流程。

在上面使用了三个关键方法 `askPermission`, `registerServiceWorker`, `subscribePush`，接下来我们分别讲述下三个方法的实现。

#### 通知权限获取

当用户打开浏览器时探测用户的浏览器是否支持 Push 的功能，然后再请求推送通知的权限。打开 `push.js`，添加下面的方法：

```
function askPermission() {
  return new Promise(function(resolve, reject) {
    // permissonResult: granted | default | denied
    var permissonResult = Notification.requestPermission(function(result) {
      resolve(result);
    });

    if (permissonResult) {
      permissonResult.then(resolve, reject);
    }
  })
  .then(function(permissonResult) {
    if (permissonResult !== 'granted') {
      throw new Error('notification denied!');
    }
  })
}
```

上面的 `askPermission` 方法会返回一个 Promise 对象，因为在推送相关的大部分 API 中都是采用的 Promise 的方式调用的，所以这里也采用同样的方式来定义， 方法本身主要就是调用了 `Notification.requestPermission` API 来请求当前通知推送的权限，这时用户会看到一个小型的浏览器弹出窗口，来确认或者拒绝通知推送，和手机端的体验基本是一样的。

授权结果保存在 `permissonResult` 中，取值有三种情况: **granted | default | denied**，我们只关心**granted**的情况，其他都为没有被授予权限。

如果用户确认的话 `askPermission` 会返回一个 Promise 对象，如果拒绝的话会抛出异常，需要在接下来的代码中捕获这个异常。待会会使用这个方法。

#### 注册 Service Worker

在第一章节中提到通知推送需要配合 Service Worker 来使用，期望的结果是即使用户没有打开我们的网站也能收到消息的推送，Service Worker 就是运行在浏览器后台的一段 JavaScript 代码，当浏览器收到推送的消息时会触发规定的事件，我们就可以在 Service Worker 中监听这些事件，然后做出处理，这个和在网页中的事件操作很相似。

打开 `push.js` 文件，添加下面的方法：

```
function registerServiceWorker(workerFile) {
  return navigator.serviceWorker.register(workerFile)
  .then(function(registration) {
    console.log('service worker registered!')
    return registration;
  })
  .catch(function(err) {
    console.error('unable to register service worker', err);
  });
}
```

`registerServiceWorker` 方法接收一个参数，这个参数为一个 JS 文件的地址，也就是在 Service Worker 中要运行的代码，比如这样来调用:

```
registerServiceWorker('/service-worker.js');
```

这个方法本身比较简单，直接调用了 `navigator.serviceWorker.register` 浏览器内置方法来注册一个 Service Worker，如果注册失败会在 Console 中打印出错误信息。

接下来就需要实现 Service Worker 的逻辑了。

#### Service Worker 实现

建立一个文件，命名为 `service-worker.js`，把这个文件放入**public**目录当中。

> 这里一定要把 **service-worker.js** 放入你项目的根路径当中，也就是 Rails 的**public**目录当中，而不能放入上面的 `app/assets/javascripts` 目录中，因为这和 Service Worker 的设计和不同浏览器的实现有关，所以为了在不同浏览器中行为一致，一定要放到你项目的根路径当中，比如 [http://localhost:3000/service-worker.js，不能放到子目录中。](http://localhost:3000/service-worker.js%EF%BC%8C%E4%B8%8D%E8%83%BD%E6%94%BE%E5%88%B0%E5%AD%90%E7%9B%AE%E5%BD%95%E4%B8%AD%E3%80%82)

首先在 `service-worker.js` 中添加下面的方法：

```
self.addEventListener('push', function(event) {
  // console.log('push event: ', event);
  var data = event.data.json();

  event.waitUntil(
    self.registration.showNotification(data.title, {
      body: data.body,
      icon: data.icon,
      tag: data.tag,
      data: {url: data.url}
    })
  );
});
```

首先监听了 `push` 事件，就是当收到推送消息时，`event` 变量中能够获得推送消息的内容本身，也就是说在发送推送时，可以指定一个 JSON 类型的消息体，然后在 `event` 中能够反向取得消息的内容。

然后调用 `waitUntil` 方法，在内部调用了 `showNotification` 方法来弹出系统的通知框，`showNotification` 方法返回的同样是一个 Promise 对象，在 `showNotification` 方法中可以指定通知框的具体内容，这里直接根据 `event` 中的数据来显示通知内容，第一个参数为标题，第二个参数为一个对象：

- body: 通知框内容
- icon: 在通知框中显示的图标，这个一般可以指定为一个 URL 地址
- tag: 通知分类信息，这个可以根据自己的实际情况随便设置就行
- data: 这个为自定义的属性，可以在这个属性中放置需要的自定义信息，比如这里可能需要传递当前信息的 URL 地址

**因为 Service Worker 为由浏览器控制的后台任务，这里采用 `waitUntil` 方法是用来把 `showNotification` 方法放入浏览器的任务队列当中**，如果这样:

```
self.addEventListener('push', function(event) {
  // console.log('push event: ', event);
  var data = event.data.json();

  self.registration.showNotification(data.title, {
    body: data.body,
    icon: data.icon,
    tag: data.tag,
    data: {url: data.url}
  })
});
```

是不行的，一定要注意。

当通知显示的时候，用户是可以和通知框进行交互的，比如点击通知内容，或者关闭通知，当点击通知内容时就可以打开对应的网页了，因此需要来处理点击的事件。

再次打开 `service-worker.js`，添加下面的代码：

```
self.addEventListener('notificationclick', function(event) {
  event.notification.close();

  event.waitUntil(clients.matchAll({
    type: 'window'
  }).then(function(clientList) {
    if (event.notification.data && event.notification.data.url) {
      return clients.openWindow(event.notification.data.url);
    } else {
      return clients.openWindow("https://eggman.tv");
    }
  }));
});
```

上面的方法监听了 `notificationclick` 事件，也就是通知点击事件，同样可以在回调方法的 `event` 参数中取得当前通知的消息内容，这里主要是取出了自定义数据 `data` 中的 url，然后再控制浏览器打开 url。比如推送了一篇最近技术文章的消息，同时在消息中传递了文章的 URL，然后这里就可以取得这个 URL，当用户点击通知时就能直接打开对应的页面了。

这里根据 `data` 是否存在做了不同的处理，因为在 Firefox 中，data 属性有丢失的情况。

#### 获取推送的 subscription 信息

我们已经获取到了推送的权限，注册了 Service Worker 来处理推送的消息，接下面最关键的问题是怎样给用户发送推送呢，这就是最后一个方法 `subscribePush` 了。

重新打开 `push.js` 文件，添加下面的代码：

```
function urlBase64ToUint8Array(base64String) {
  var padding = '='.repeat((4 - base64String.length % 4) % 4);
  var base64 = (base64String + padding).replace(/\-/g, '+').replace(/_/g, '/');

  var rawData = window.atob(base64);
  var outputArray = new Uint8Array(rawData.length);

  for (var i = 0; i < rawData.length; ++i) {
      outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}

function subscribePush(registration) {
  var subscribeOptions = {
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(AppConfig.PUBLIC_SERVER_KEY)
  };
  return registration.pushManager.subscribe(subscribeOptions)
  .then(function(pushSubscription) {
    console.log('push subscription: ', JSON.stringify(pushSubscription));

    return pushSubscription;
  });
}
```

我们添加了两个方法，第一个 `urlBase64ToUint8Array` 是用来把一个 base64 的字符串转化为整数数组的，因为根据 Web Push 协议的要求需要把**public key**来转化整数数组才能发送到推送中心服务器，这个方法就不多说了。

关键是 `subscribePush` 方法，在这个方法中调用了 `registration.pushManager.subscribe` 方法，其中 `registration` 是在 `load` 中 Service Worker 注册成功后返回的对象，`registration.pushManager.subscribe` 就是用来注册并返回当前浏览器的推送的 subscription 信息，这个方法需要传递一个参数，也就是 `subscribeOptions`。

subscribeOptions:

- userVisibleOnly: 通知用户是否课件，目前 Chrome 和 Firefox 只支持可见的通知，也就是这个参数必须为 true
- applicationServerKey: 这个就是在第一章中配置的 public key 了，这里注意要使用 `urlBase64ToUint8Array` 方法转化为整数数组。

调用 `registration.pushManager.subscribe` 方法简单来说就是向推送的中心服务器注册并取得当前浏览器的推送识别参数，就是和当前浏览器绑定的全局唯一的一组 Hash，我们称为 subscription 信息，也就是上面的 `pushSubscription` 变量。`pushSubscription` 的结构为这样的：

```
 {
   "endpoint": "https://fcm.googleapis.com/fcm/send/cBBFXqYE6Ew:APA91bFKMLXp9lDwbjiZxjLIRdl1CjKUeDwXljGqdaEmdVdywdntFsPZE1KMeeDr13YIsaAzj2up8fwHTWCjsongLxdxb_vsCdVk",
   "keys":{
     "p256dh": "BHAPGwdf4j81Z_nZtgaHhoXpLFq-EvfM1aZGyhHrdbj2loRsXwm0_7Box0jCR8oXtsIRjh4F3zvw9uzdyddh-9A=",
     "auth": "y7PfWaVteugvp8C37NzA5Q=="
   }
 }
```

这个数据待会在测试中就能看到。

目前为止我们已经完成了所有前端工作，简单来说就是两个 JS 文件：`app/assets/javascripts/push.js` 和 `public/service-worker.js`。

### 后端调整

然后就是怎样在页面中引入 `push.js` 文件了，打开 `app/assets/javascripts/application.js.erb`，删除其中的：

```
 //= require_tree .
```

这行代码，最终的 `app/assets/javascripts/application.js.erb` 为这样：

```
 //= require rails-ujs

window.AppConfig = {
  PUBLIC_SERVER_KEY: '<%= WEB_PUSH_PUBLIC_KEY %>',
}
```

打开 `config/initializers/assets.rb` 文件，修改 `precompile` 行的配置为这样：

```
Rails.application.config.assets.precompile += %w(
  push.js
)
```

打开 `app/views/layouts/application.html.erb` 文件，修改为：

```
<!DOCTYPE html>
<html>
  <head>
    <title>WebPushSample</title>
    <%= csrf_meta_tags %>
    <%= stylesheet_link_tag    'application', media: 'all' %>
    <%= javascript_include_tag 'application' %>
  </head>

  <body>
    <%= yield %>
    <%= yield :js %>
  </body>
</html>
```

最后再打开 `app/views/welcome/index.html.erb`，修改为:

```
<h1>Web Push Notification</h1>
<%= content_for :js do %>
    <%= javascript_include_tag "push" %>
<% end %>
```

### 测试和发送推送

重启你的 `rails s` 进程，打开浏览器的开发者工具中的 console，访问**localhost:3000**，这时你应该就能看浏览器弹出通知推送的窗口:

![img_alt](https://imgs.eggman.tv/1499054169948469Screen-Shot-2017-07-03-at-11-55-39-AM-png.png)

点击同意之后，在 console 中就能看到打印出的 subscription 信息了，结构和上面提到的 `pushSubscription` 一样:

```
 {
   "endpoint": "https://fcm.googleapis.com/fcm/send/cBBFXqYE6Ew:APA91bFKMLXp9lDwbjiZxjLIRdl1CjKUeDwXljGqdaEmdVdywdntFsPZE1KMeeDr13YIsaAzj2up8fwHTWCjsongLxdxb_vsCdVk",
   "keys":{
     "p256dh": "BHAPGwdf4j81Z_nZtgaHhoXpLFq-EvfM1aZGyhHrdbj2loRsXwm0_7Box0jCR8oXtsIRjh4F3zvw9uzdyddh-9A=",
     "auth": "y7PfWaVteugvp8C37NzA5Q=="
   }
 }
```

如果你是在 Firefox 中测试的话，会看到**endpoint**的 URL 地址会不一样，这也就是我们提到的不同浏览器的中心服务器不一样，大家只用把这个信息拷贝一下，发送推送就是使用的这个信息，接下来就可以测试了：

进入 `rails c` 中：

```
> message = {
  title: "eggman.tv 通知",
  body: "web push notifications",
  icon: "https://eggman.tv/images/web-push-icon.png",
  url: "https://eggman.tv"
}
> sub_data = {
  endpoint："https://fcm.googleapis.com/fcm/send/c170mQtn8y4:APA91bH4pT6po7s0IJ8yFbYcw4Xl07qv8Kxl3muhUxwnleL9ShBSUwrrRbGXylK9StOhvWFBJy1oUo0f_0Ky0SklHgf388K2yrBnV4stdnuvD-DksJwQbzFBRL_LiP1ba86eX7li9-E",
  keys： {
    p256dh： "BBeXsuEI7lv1zg89bVDRLmWE257Rb6a6Y3iv0rNrbSILLtAmDt17CwB-zywVztjPJupnPxd0E9cJhI7ldjpTSQ=",
    auth： "ijEK4vIGJ3vDCeeJbopDtg=="
  }
}
> Webpush.payload_send(
  endpoint: sub_data[:endpoint],
  message: message.to_json,
  p256dh: sub_data[:keys][:p256dh],
  auth: sub_data[:keys][:auth],
  vapid: {
    subject: 'service@eggman.tv',
    public_key: WEB_PUSH_PUBLIC_KEY,
    private_key: WEB_PUSH_PRIVATE_KEY
  }
)
```

上面的 `message` 为发送的测试信息，`sub_data` 为刚刚在浏览器 console 中复制的 subscription 信息，最后调用 `webpush` gem 的 `payload_send` 方法来发送推送，参数都是比较直观的，其中 `vapid.subject` 需要输入一个 Email 地址，这也是 Web Push 协议要求的，大家可以随便填写一个 email 就可以了。

运行完上面的代码你应该就已经成功收到浏览器的推送消息了。

### Debug

JavaScript 的调试可以在 Chrome 的开发者工具中进行，打断点，这个就不再多做介绍了，这里主要是 Service Worker 的调试给大家介绍一下。

在 Chrome 的开发者工具中可以点击**Application**的 tab，然后选择左侧菜单中的**Application - Service Worker**来查看当前在该域名下注册的所有的 Service Worker，以及进行一些管理：

![img_alt](https://imgs.eggman.tv/1499054596889078Screen-Shot-2017-07-03-at-12-02-53-PM-png.png)

在开发过程中可以点击右侧的**Unregister**按钮来删除 Service Worker，然后刷新页面重新注册。

我们已经实现了消息的推送功能，从上面的代码可以看出需要把 subscription 的信息保存在数据库当中，之后发送推送消息时需要使用。这方面的内容涉及到后端的具体实现，如果你对这方面感兴趣，就请继续往下看吧，点击这里观看第三部分：[Web Push Notifications: 保存推送的subscription信息](https://eggman.tv/lessons/web-push-notifications-backend-logic-development)

## Web Push Notifications: 保存推送的 subscription 信息

这里的内容是由像和你一样的工程师们一行一行创造出来的，为了保护你的权益，同时也为了激励我们继续创造更好的内容，蛋人网所有内容禁止一切形式的转载和复制

> 这篇文章是 [**Web Push Notifications**](https://eggman.tv/c/s-web-push-notifications) 系列文章的第三部分，如果要观看第二部分请点击这里: [Web Push Notifications: 通知推送订阅功能开发](https://eggman.tv/lessons/web-push-notifications-subscription-development)

在上一章节中我们已经取得了推送的 subscription 信息：

```
 {
   "endpoint": "https://fcm.googleapis.com/fcm/send/cBBFXqYE6Ew:APA91bFKMLXp9lDwbjiZxjLIRdl1CjKUeDwXljGqdaEmdVdywdntFsPZE1KMeeDr13YIsaAzj2up8fwHTWCjsongLxdxb_vsCdVk",
   "keys":{
     "p256dh": "BHAPGwdf4j81Z_nZtgaHhoXpLFq-EvfM1aZGyhHrdbj2loRsXwm0_7Box0jCR8oXtsIRjh4F3zvw9uzdyddh-9A=",
     "auth": "y7PfWaVteugvp8C37NzA5Q=="
   }
 }
```

发送推送就需要这个信息，所以现在就来保存这个信息到数据库中。

### WebPushEndpoint 模型

采用保存 subscription 信息的模型为 `WebPushEndpoint`，在项目的根目录中运行下面的命令来创建模型：

```
$ rails g model web_push_endpoint
```

然后编辑 `app/models/web_push_endpoint.rb` 文件为：

```
class WebPushEndpoint < ApplicationRecord

  belongs_to :user

  def self.find_or_create! options
    unless record = find_by(endpoint: options[:endpoint])
      record = create! user: options[:user],
        endpoint: options[:endpoint],
        subscription: options.except(:user).to_json
    end

    record
  end

end
```

在模型中定义了 `find_or_create!` 方法，这个方法判断了 endpoint 是否存在，如果不存在会根据传递进来的 subscription 信息来创建新的记录。这里有一点要注意的是我们把 subscription 信息转化为了 JSON 字符串保存在了 subscription 字段中。我们会在 controller 中调用这个方法。

对应的数据库移植为：

```
class CreateWebPushEndpoints < ActiveRecord::Migration[5.1]
  def change
    create_table :web_push_endpoints do |t|
      t.integer :user_id
      t.string :endpoint
      t.text :subscription
      t.timestamps null: false
    end

    add_index :web_push_endpoints, [:user_id]
    add_index :web_push_endpoints, [:endpoint], unique: true
  end
end
```

字段： **user_id**：用来管理系统中的用户模型 **endpoint**: subscription 信息中的 `endpoint` URL，因为 subscription 信息和浏览器是绑定的，endpoint URL 是全局唯一的，我们把这个作为模型的唯一主键。用户每次在打开浏览器时都会发送一遍 subscription 信息，此时只有在 endpoint URL 不存在的情况下才会创建新记录，同一个浏览器每次发送的 subscription 信息是一样的，除非当用户取消授权的情况下才会发生变化。 **subscription**: 这个字段用来存储整个 subscription 信息，当然这里 endpoint URL 是作为冗余信息再存储一遍。

### 发送 subscription 信息到后端

`subscription` 是在调用 `subscribePush` 这个方法时获取的，修改 `push.js` 文件中的的这个方法为：

```
function subscribePush(registration) {
  var subscribeOptions = {
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(AppConfig.PUBLIC_SERVER_KEY)
  };
  return registration.pushManager.subscribe(subscribeOptions)
  .then(function(pushSubscription) {
    fetch('/web_push_endpoints',
      {method: 'POST', body: JSON.stringify(pushSubscription)})
    .then(function(response) {
      if (response.ok == true) {
        response.json().then(function(data){
          if (data.status == 'ok') {
            console.log('push subscription: ', JSON.stringify(pushSubscription));
          }
        })
      }
    });

    return pushSubscription;
  });
}
```

这里采用 HTML5 的 `fetch` 方法把获取到的 subscription 信息发送到后端的 `/web_push_endpoints` URL，实现逻辑是比较简单的，就不再详细解释了。

然后添加对应的 controller 逻辑，首先修改 `config/routes.rb` 文件，添加路由：

```
resources :web_push_endpoints, only: [:create]
```

然后在 `app/controllers` 目录下面创建对应的 `web_push_endpoints_controller.rb`:

```
class WebPushEndpointsController < ApplicationController

  skip_before_action :verify_authenticity_token, only: [:create]

  def create
    data = request.body.read
    subscription = JSON.parse(data)
    subscription.deep_symbolize_keys!
    subscription.merge!(user: current_user) if logged_in?

    WebPushEndpoint.find_or_create! subscription
    render json: {status: 'ok'}
  end

end
```

上面调用了模型 `WebPushEndpoint` 中的 `find_or_create!` 方法来创建 subscription 信息。

至此我们已经完成了后端 subscription 信息的创建功能，大家可以重新刷新一下前端页面，应该可以看到后台的 web_push_endpoints 表中新增了一条数据记录，然后发送推送信息时就可以从 WebPushEndpoint 模型中读取了。

### 发送推送

批量发送推送的话就可以这样：

```
WebPushEndpoint.find_each do |web_push_endpoint|
  sub_data = JSON.parse web_push_endpoint.subscription

  message = {
    title: options[:title],
    body: options[:body],
    icon: options[:icon],
    tag: options[:tag],
    url: options[:url],
  }
  Webpush.payload_send(
    endpoint: sub_data['endpoint'],
    message: message.to_json,
    p256dh: sub_data['keys']['p256dh'],
    auth: sub_data['keys']['auth'],
    vapid: {
      subject: 'service@eggman.tv',
      public_key: WEB_PUSH_PUBLIC_KEY,
      private_key: WEB_PUSH_PRIVATE_KEY
    }
  )
end
```

在线上的环境中可以结合一些异步任务组件比如**Sidekiq**来实现批量的异步发送，这里就不再深入说明了，想了解**Sidekiq**使用的同学可以点击这里继续观看: [Sidekiq的使用](https://eggman.tv/c/s-learn-ruby-common-gems)

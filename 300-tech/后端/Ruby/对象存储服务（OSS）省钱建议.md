---
title: 对象存储服务（OSS）省钱建议
date: 2024-08-16T10:00:00+08:00
updated: 2024-08-21T10:32:33+08:00
dg-publish: false
source: //ruby-china.org/topics/41989
---

这篇文章简单谈谈对象存储服务的省钱建议，在创业公司干活，成本控制还是有点重要的，毕竟没有金主爸爸照顾，每个月省点钱，说不定就能“活”久一点。十分感谢 Ruby China 会员 [哥有石头](https://ruby-china.org/jicheng1014) 的指点。原文连接： [https://lanzhiheng.com/posts/save-money-in-oss](https://lanzhiheng.com/posts/save-money-in-oss)

---

## 对象存储服务（OSS）

OSS 全称是 Object Store Service（对象存储服务）。这么说可能有点抽象。可以简单把它理解成一个托管资源的地方，这些资源也就是名称中的“对象”。运营网站的过程中总是少不了要存储，图片，小视频这些静态资源。如果是在服务器本机存储这些东西，维护成本太高，不利于迁移，并且容灾效果不佳。一般会建议“不要把鸡蛋放在同一个篮子里”，也就是托管到第三方的服务中去，而这个存储资源的服务名为对象存储服务。

如果只有存储功能，那么 OSS 服务就更硬盘没啥区别了，它还必须要有外网访问的能力，这样你才能把东西上传上去（上行流量），并且把上传的东西供给别人访问/下载（下行流量）。简单来说 OSS 就是集存储与访问于一身的服务。还能够根据需要对自己的资源进行备份以免数据丢失。

## 费用分析

科技发展到今天这个地步，磁盘空间几乎已经成了最廉价的资源。故而在 OSS 中，存储费用其实是很低的。而在流量为王的年代，最贵的还得是流量。而对于任何服务来说基本都会是上行量要远小于下行量。故而下行流量也成了 OSS 中最为昂贵的一环。笔者公司上个月 OSS 服务的上行流量也就几十 G，而下行流量是 3.2TB 左右。下行流量换算成人民币大概是 1600 元，单价是 0.5 元/GB。当流量只有几百 G 的时候你可能感觉不到有啥，然而当流量上 T 之后，多少也会感到不安。按照网站如今的访问量，下个月的流量保守估计能到 5T。这笔费用想想就有点害怕。

当时还是阿里云的双 11 活动，我们这边有两套方案

1. 购买阿里云的 OSS 下行流量包，按年购买每个月 5TB 的流量，大概 1.6W 左右的费用，费用大概是 0.26 元/GB。
2. 不再使用 OSS 下行流量的方式直接读取资源。而是采用 CDN 回源的形式，下行流量统一走 CDN 流量，费用大概是 0.24 元/GB ~ 0.26 元/GB 这样。

方案一比较省事，趁着双十一直接掏钱就完了，然而可扩展性极差，万一哪个月流量超 5TB，那么多出来的那部分依旧需要按照 0.5 元/GB 来计费，从长远来看并不是一个很好的选择，这么便宜的流量包也不是每个月都能买到。方案二则比较费事，需要配置 CDN 域名，CDN 访问 OSS 的权限，SSL 证书等等，不熟悉的话可能要折腾个大半天，然而长远来看不管每个月流量多少，都是按照 0.24 元/GB ~ 0.26 元来计算，果断选择方案二。

所谓 CDN 回源，指的是当 CDN 服务并没能缓存到对应资源的时候需要回源到 OSS 端去下载。用户端访问 CDN 服务，CDN 服务找不到这个资源则再去 OSS 那边缓存资源到本地，所产生的费用跟用户端直接访问 OSS 获取资源是完全不同的。前者产生的叫回源流量，后者产生的是 OSS 下行流量。OSS 下行流量一般比较贵，量也比较大，回源流量一般量不会很大，CDN 有缓存之后都是直接从 CDN 返回，CDN 下行流量费用也相对便宜。

虽说方案二除了 CDN 下行流量费用，还需要支付 HTTPS 费用（按照次数），以及回源流量费用，不过后面两者相对比较便宜，不管怎么算都比以“公共读”的方式从 OSS 直接获取资源要便宜许多。虽说方案二比较费事，但是费用降低了接近一半。假设每个月流量能达到 5TB 左右，粗略估计大概

直接从 OSS 上读取：5 x 1024GB x 0.5 元/GB = 2560 元 CDN 回源优化之后：5 x 1024GB x 0.26 元/GB（CDN 下行流量是 0.24 元/GB，再加上回源流量，HTTPS 请求次数的费用，粗略按照 0.26 元/GB 计算）= 1331 元

虽说不是很精确，但保守估计这个月能省个 1000 块钱左右吧，流量越大省的越多。

## 更进一步

上面是从云服务商本身做的优化，从 OSS 直连下载的模式切换到 CDN 回源到 OSS 的模式，原则上能省下一大笔。并且，还可以在此基础上做更多的优化工作

## 购买流量包

一般来说购买流量包的流量费用要比按量付费要便宜。这里已经从 OSS 下行流量切换到 CDN 流量了，所以需要购买 CDN 下行流量包。OSS 的下行量流量包有点像中国移动的模式，购买流量包之后每个月都有定额流量，超额了要另外购买，用不完的就浪费掉了。而 CDN 的下行流量是，有效期是一年，用多少买多少，假设你买了 5T 的流量，用不完，下个月继续用，用完了再买就好，个人会更喜欢 CDN 这种模式。CDN 的下行流量包，笔者所购买的是 5T 的套餐，大概是 768 元，这样算下来单价是 0.15 元/GB。比起原来的 0.5 元/GB 的 OSS 下行费用，一个月下来要轻松不少呢。

[](https://step-by-step.oss-cn-beijing.aliyuncs.com/production/63cw8k4niin797j6yas34laucpbw)

[![套餐.png](https://step-by-step.oss-cn-beijing.aliyuncs.com/production/63cw8k4niin797j6yas34laucpbw)

](<https://step-by-step.oss-cn-beijing.aliyuncs.com/production/63cw8k4niin797j6yas34laucpbw>)

## 节流

后来发现公司自身的资源也有很大的优化空间，这方面不同的公司可能不一样。笔者的公司主要流量都是图片，视频。而其中的流量大头其实是视频。发现最开始用 iPhone 拍摄的视频大概是 20~30M 左右。这不仅上传耗时，用户下载视频也费流量。后来我们决定采用 720P 的配置来拍摄视频，视频大小大概缩小到 17M 左右，又因为用户是用手机预览的视频，所以几乎不会感觉到区别。

在服务层面，再对原来 iPhone 拍摄出来的 MOV 格式的视频做压缩并转换成 MP4 格式，大小能够进一步减少，而且 MP4 格式的兼容性会相对好一些。转换后的视频大小大概是 2MB ~ 5MB 左右。以下是我的视频转换脚本，如果不执着于无损压缩，用这个配置去压缩的视频一般人肉眼也分辨不出来

```
ffmpeg -i source.mov -vcodec h264 -y -preset veryfast target.mp4
```

视频成功转换并上传到云服务之后，转换后的视频依旧会保存在服务器，如果放着不管，一段时间之后磁盘必然会塞爆。这个时候做个定时任务处理一下即可

```
# app/jobs/auto_clean_videos_directory_job.rb
class AutoCleanVideosDirectoryJob < ApplicationJob
  queue_as :default

  def perform(*_args)
    FileUtils.rm Dir.glob(Rails.root.join('tmp/videos/*'))
  end
end
```

```
# config/schedule.yml

auto_clean_videos_directory_job:
  cron: "0 0 */10 * *"
  class: "AutoCleanVideosDirectoryJob"
  queue: :daily
```

倒也不用太频繁，每 10 天清除一遍遗留下来的视频数据就可以了。如果磁盘剩余空间小，可以考虑把频次提高一些。

## 尾声

这篇文章简单总结了一下笔者近期在优化 OSS 费用方面所做的工作。目前看来比较经济的策略是 OSS 只做存储，下行流量走 CDN，再搭配上流量包，流量的费用能够节约不少。虽然说这种做法会产生回源流量，不过总的来说回源流量也要比 OSS 下行流量要便宜不少。而在自身服务层面，则可以尽可能降低资源的大小。咋一看从 15M 压缩到 5M 只少了 10M 似乎没什么，然而要是这个视频有 500 个人访问，那其实就相当于节省了 5G 左右的流量了。长远来看还是有点价值的。

---

和我这里的方案差不多。 个人建议，当你的下行流量比较大时，且未来会持续增大，可以去跟云服务厂商谈价格，大云服务厂商不好谈，中小型容易谈。可以谈到比买流量包还便宜。

---

还可以在 oss 上分 热资源 冷资源 继续压缩成本。。。

---

我自己用都是，云主机 Nginx 反代到 OSS，缓存时间搞大，这样算内网流量了，白嫖

哦？ 您这么一说。好像有一定可搞性。 先记下来。

---

其实 cdn 你也可以设置缓存时间，之后访问的时候如果用户之前访问过，就会单纯的 304。

如果用户没有访问过的话，无论是 cdn 还是 nginx 反代，都会走服务器的传输带宽（对应用户的下载），如果你的出口带宽是 弹性 ip 计费，则跟 oss 直接下载到用户是同价的

---

如果不怎么差钱的可以采用一些标准云计算方案：

1. 客户端直接上传到 OSS，不经过服务端。
2. 使用转码服务，将新上传的视频做多码率压缩（1080->720/480）和抽抽帧/视频高光摘要生成。
3. 配置 OSS 文件存储策略（OSS 文件前缀），比如 7 天前数据滚动删除，3 天前的数据转冷存储。
4. CDN 可以根据业务模式选择流量/峰值带宽等收费方式。

---

如果使用阿里云 OSS，转码服务可以使用阿里云的“媒体处理 MPS”。

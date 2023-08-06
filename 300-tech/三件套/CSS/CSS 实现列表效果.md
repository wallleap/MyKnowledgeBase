---
title: CSS 实现列表效果
date: 2023-08-03 19:29
updated: 2023-08-03 19:29
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - CSS 实现列表效果
rating: 1
tags:
  - CSS
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



到今天，我们的【每日一练】系列内容来到了第二个100练，在第一个100练的时候，我给自己定了一个小目标，就是分享1000个练习，在这前面的200练中，我基本是想到什么就分享什么，也没有什么规律可言，也没有特意安排今天要学习什么内容，练习什么内容，都是属于随意性的。  

因为，我真的觉得，练习是学编程中非常重要的一个环节，如果不写练习的话，真的不可能把编程学会。  

当然，我们写了练习，也不一定能够把编程学好。

如果是这样的话，那我为什么还要写练习呢？

这个问题就好比，我们每天都会吃饭，但是我们每天吃了什么菜，你肯定不会全部都记得，但是，吃了健康的食物，肯定会成为我们身体持续运转的能量。  

学编程也是如此，只有不断地练习，不断输入，我们才能在需要地时候快速输出。

另外，就是从明天开始，也就是第201练开始，我不会在每日更新一个内容了，我会做专题内容进行练习，更新地内容与数量会有所增加，因此，更新频次会有所调整。

**后面，【每日一练】栏目固定更新时间与位置：**

**每周一第二条更新一次【每日一练】栏目地专题内容，请大家注意关注。**

因此，从明天开始，【每日一练】栏目内容没有更新，我会在下周一统一更新第一个专题内容。

当然，因为内容篇幅地关系，我也不会在文章中放置源码了，但是，我依然会把代码打包好，供大家学习使用。  

好了，今天就先跟大家汇报这么多了，有什么问题，欢迎在留言去给我留言，我看到后，会第一次回复。  

现在，我们还是一起来看一下今天练习的最终效果：

![](https://cdn.wallleap.cn/img/pic/illustration/202308031929825.gif)

HTML代码：  

```html
<htm>
    <head> 
        <meta charset="UTF-8"> 
        <title>【每日一练】200—CSS实现每日一练内容的列表效果</title> 
        <link href="style.css" rel="stylesheet" type="text/css"> 
    </head>
    <body>
        <div class="container">
            <div class="wrapper">
              <h1>【web前端开发】公号平台每日一练全集</h1>
              <ul class="sessions">
                <li>
                  <div class="time">2023-6-14</div>
                  <p><a href="#" target="_blank">【每日一练】200—CSS实现3D菜单效果</a></p>
                </li>
                <li>
                  <div class="time">2023-6-13</div>
                  <p><a href="#" target="_blank" >【每日一练】199-CSS实现发光按钮Hover 效果</a></p>
                </li>
                <li>
                  <div class="time">2023-6-12</div>
                  <p><a href="#" target="_blank">【每日一练】198—实现动画循环进度</a></p>
                </li>
                <li>
                  <div class="time">2023-6-13</div>
                  <p><a href="#" target="_blank">【每日一练】197—CSS 创意菜单栏的文本动画效果</a></p>
                </li>
                <li>
                  <div class="time">2023-6-14</div>
                  <p><a href="#" target="_blank">【每日一练】196—CSS 实现创意按钮动画效果</a></p>
                </li>
                <li>
                  <div class="time">2023-6-15</div>
                  <p><a href="#" target="_blank">【每日一练】195—实用又有创意的产品卡片动画</a></p>
                </li>
                <li>
                  <div class="time">2023-6-16</div>
                  <p><a href="#" target="_blank">【每日一练】194—CSS实现响应式产品介绍的Hover效果 </a></p>
                </li>
                <li>
                  <div class="time">2023-6-17</div>
                  <p><a href="#" target="_blank">【每日一练】193—404页面的动画效果</a></p>
                </li>
                <li>
                  <div class="time">2023-6-18</div>
                  <p><a href="#" target="_blank">【每日一练】192—透明列表Hove效果的实现</a></p>
                </li>
                <li>
                  <div class="time">2023-6-19</div>
                  <p><a href="#" target="_blank">【每日一练】191—移动3D触摸滑块的实现</a></p>
                </li>
                <li>
                  <div class="time">2023-6-20</div>
                  <p><a href="#" target="_blank">【每日一练】190—复选框效果的实现</a></p>
                </li>
              </ul>
            </div>
          </div> 
    </body>
</htm>
```

因为篇幅的关系，我就不把全部内容加进来。

其实，写静态的HTML页面的时候，就是这样，同样的效果，复制相同代码即可，所以，如果要补全，也是很快的，Ctrl+C；Ctrl+V就好了，大家可以直接尝试一下。

SCSS 代码：

```scss
@mixin tablet-and-up {
    @media screen and (min-width: 769px) { @content; }
}
@mixin mobile-and-up {
    @media screen and (min-width: 601px) { @content; }
}
@mixin tablet-and-down  {
    @media screen and (max-width: 768px) { @content; }
}
@mixin mobile-only {
    @media screen and (max-width: 600px) { @content; }
}

ul, li{
  list-style: none;
  padding: 0;
}

.container{
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0 1rem;
  background: linear-gradient(45deg, #209cff, #68e0cf);
  padding: 3rem 0;
}
.wrapper{
  background: #eaf6ff;
  padding: 2rem;
  border-radius: 15px;
}
h1{
  font-size: 1.1rem;
  font-family: sans-serif;
}
.sessions{
  margin-top: 2rem;
  border-radius: 12px;
  position: relative;
}
li{
  padding-bottom: 1.5rem;
  border-left: 1px solid #abaaed;
  position: relative;
  padding-left: 20px;
  margin-left: 10px;
  &:last-child{
    border: 0px;
    padding-bottom: 0;
  }
  &:before{
    content: '';
    width: 15px;
    height: 15px;
    background: white;
    border: 1px solid #4e5ed3;
    box-shadow: 3px 3px 0px #bab5f8;
    box-shadow: 3px 3px 0px #bab5f8;
    border-radius: 50%;
    position: absolute;
    left: -10px;
    top: 0px;
  }
}
.time{
  color: #2a2839;
  font-family: 'Poppins', sans-serif;
  font-weight: 500;
  @include mobile-and-up{
    font-size: .9rem;
  }
  @include mobile-only{
    margin-bottom: .3rem;
    font-size: 0.85rem;
  }

}
p{
  color: #4f4f4f;
  font-family: sans-serif;
  line-height: 1.5;
  margin-top:0.4rem;
  @include mobile-only{
    font-size: .9rem;
  }
}
p a{
    color:#222;

}
p a:hover{
    color:#bab5f8;

}
```

**写在最后**

以上就是今天【每日一练】的全部内容，希望今天的小练习对你有用，如果你觉得有帮助的话，请点赞我，关注我，并将它分享给你身边做开发的朋友，也许能够帮助到他。

我是杨小爱，我们下期见。

**推荐阅读**

[【每日一练】01~100练大合集汇总](http://mp.weixin.qq.com/s?__biz=MjM5MDA2MTI1MA==&mid=2649133984&idx=1&sn=0d274ff12af27e618423d822ed7119d2&chksm=be58b40d892f3d1b6d3b7a0f2e57cb16fc6e774282512517925d28ab01e5d009863f753c4590&scene=21#wechat_redirect)

视频号上的单页网站模板的源码，大家可以到我们博客网站上去进行获取，地址如下：http://www.webqdkf.com/archives/832


---
title: viewport 移动端适配
date: 2023-08-04 17:25
updated: 2023-08-04 17:26
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - viewport 移动端适配
rating: 1
tags:
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

提到 viewport 移动端适配，那么心中可能有几个疑问？

1.为什么要进行移动端的适配?

2.viewport 是什么?

3.如何进行移动端适配?

## 一、移动端适配的目的

通常我们在 pc 上看到的页面都是比较大的，在 pc 上访问页面是正常显示的，默认是不会被缩放的，除非是手动进行了缩放，页面才会被放大比例或者是缩小比例显示。但是在移动端是不一样的，如果将一个 pc 端的页面放到手机端进行访问，那么可能出现页面挤到一起、布局错乱或者出现横向滚动条的情况，我们给用户带来不好的体验。还有在屏幕尺寸大小不同的手机上进行访问页面时，页面显示的效果不能合理的展示，我们期望的是在手机屏幕较大时显示的内容比较大一些，手机屏幕小的时候页面的内容也会缩小进行自适应。

因此，移动端适配的目的是在不同尺寸的设备上，页面达到合理的展示（自适应）或者说是能够保持统一效果。

## 二、viewport 的基本概念

1.通常 viewport 是指视窗、视口，浏览器上 (也可能是一个 app 中的 webview) 用来显示网页的那部分区域。在移动端和 pc 端视口是不同的，pc 端的视口是浏览器窗口区域，而在移动端有三个不同的视口概念：布局视口、视觉视口、理想视口

- 布局视口：在浏览器窗口 css 的布局区域，布局视口的宽度限制 css 布局的宽。为了能在移动设备上正常显示那些为 pc 端浏览器设计的网站，移动设备上的浏览器都会把自己默认的 viewport 设为 980px 或其他值，一般都比移动端浏览器可视区域大很多，所以就会出现浏览器出现横向滚动条的情况

- 视觉视口：用户通过屏幕看到的页面区域，通过缩放查看显示内容的区域，在移动端缩放不会改变布局视口的宽度，当缩小的时候，屏幕覆盖的 css 像素变多，视觉视口变大，当放大的时候，屏幕覆盖的 css 像素变少，视觉视口变小。

- 理想视口：一般来讲，这个视口其实不是真是存在的，它对设备来说是一个最理想布局视口尺寸，在用户不进行手动缩放的情况下，可以将页面理想地展示。那么所谓的理想宽度就是浏览器（屏幕）的宽度了。

为了理解起来更清楚一点，在网上找了两张比较易懂的图片。如果对这三个视口的概念还不是很清楚，看看下面几张图可能就会一目了然：

布局视口

![](https://cdn.wallleap.cn/img/pic/illustration/202308041726553.jpeg)

视觉视口

![](https://cdn.wallleap.cn/img/pic/illustration/202308041726554.jpeg)

2.那么如何设置理想视口呢？很简单，只需要在 html 的 header 中加入一段很重要的代码

```
 <meta name="viewport"content="width=device-width,user-scalable=no,initial-scale=1.0,  maximum-scale=1.0,minimum-scale=1.0">
```

这行代码大家应该都不陌生，或者说是知道加了这行代码，然后页面的宽度就会跟我的设备宽度一致。，实际上，它就是设置了理想视口，将布局视口的宽度设置成了理想视口（浏览器/设备屏幕的宽度）。

上面说到的视口宽度等均是 css 像素，所以需要简单了解一下几个基本的概念：

- css 像素：代码中使用的逻辑像素，衡量页面上的内容大小

- 设备像素：即物理像素，控制设备显示的单位，与设备、硬件有关

- 设备独立像素：与设备无关的逻辑像素，不同于设备像素（物理像素），不是真实存在的。

- 设备像素比：定义设备像素与设备独立像素比的关系 window.devicePixelRatio）设备像素比=物理像素/设备独立像素

- 分辨率：指的是屏幕上垂直和水平的总物理像素

3.在 meta 标签中，除了 viewport 这个很重要的属性，用来设置视口的一些行为,还有几个与其搭配一起使用的属性：

| 属性 | 含义 | 取值 |
| --- | --- | --- |
| width | 定义视口的宽度，单位为像素 | 正整数或设备宽度 device-width |
| height | 定义视口的高度，单位为像素 | 正整数或 device-height |
| initial-scale | 定义初始缩放值 | 整数或小数 |
| minimum-scale | 定义缩小最小比例，它必须小于或等于 maximum-scale 设置 | 整数或小数 |
| maximum-scale | 定义放大最大比例，它必须大于或等于 minimum-scale 设置 | 整数或小数 |
| user-scalable | 定义是否允许用户手动缩放页面，默认值 yes | yes/no |

## 三、适配的几种方案

（1）css3 媒体查询: 通过媒体查询的方式，编写适应不同分辨率设备的的 css 样式

```
@media screen and (max-width: 320px){
    ....适配iphone4的css样式
}
@media screen and (max-width: 375px){
     ....适配iphone6/7/8的css样式
}
@media screen and (max-width: 414px){
    ....适配iphone6/7/8 plus的css样式
}
......
```

优点：

- 方法简单，只需修改 css 文件

- 调整屏幕宽度时不用刷新页面就可以响应页面布局

缺点：

- 代码量大，不方便维护

- 不能够完全适配所有的屏幕尺寸，需要编写多套 css 样式

(2) 百分比布局方案：给元素设置百分比，例如 2 个 div 想占满宽度 100%，那么一个 div 设置宽度为 50%，这样不固定宽度，使得在不同的分辨率下都能达到适配

那么需要清楚一个问题，各个子元素或属性的百分比是根据谁来设定呢？

1. 子元素 width、height 的百分比: 子元素的 width 或 height 中使用百分比，是相对于子元素的直接父元素
2. margin 和 padding 的百分比：在垂直方向和水平方向都是相对于直接父亲元素的 width，而与父元素的 height 无关
3. border-radius 的百分比：border-radius 的百分比是相对于自身宽度，与父元素无关

优点：

- 宽度自适应，在不同的分辨率下都能达到适配

缺点：

- 百分比的值不好计算
- 需要确定父级的大小，因为要根据父级的大小进行计算
- 各个属性中如果使用百分比，相对父元素的属性并不是唯一的
- 高度不好设置，一般需要固定高度

(3)rem 方案:

- rem 单位：rem 是一个只相对于浏览器的根元素（HTML 元素）的 font-size 的来确定的单位。默认情况下，html 元素的 font-size 为 12px
- 通过 rem 来实现适配：rem 单位都是相对于根元素 html 的 font-size 来决定大小的,根元素的 font-size 相当于提供了一个基准，当页面的宽度发生变化时，只需要改变 font-size 的值，那么以 rem 为固定单位的元素的大小也会发生响应的变化。需要先动态设置 html 根元素的 font-size,再计算出其他页面元素以 rem 为固定单位的值

控制 font-size 的 js 代码

```
<script type="text/javascript">
    (function() {
        var deviceWidth = document.documentElement.clientWidth;
        deviceWidth = deviceWidth < 320 ? 320 : deviceWidth > 640 ? 640 : deviceWidth;
        document.documentElement.style.fontSize = deviceWidth / 7.5 + 'px';
    })();
</script>
```

上面的 js 代码中 deviceWidth/7.5，表示 font-size 用 deviceWidth/7.5 的值来表示，1rem 的值就是 deviceWidth/7.5，当视口容器发生变化时就可以动态设置 font-size 的大小，不论页面宽度变大还是缩小，视口宽度都会被等分为 7.5 份，每一份就是 1rem,从而 1rem 在不同的视觉容器中表示不同的大小，但在视口总宽度中的占比是不变的，实现了等比适配。

这个 7.5 在这里并不是一个固定的值，也可以设置为其他值，因为设计稿一般是根据 iphone6/7/8 的宽度来设计，一般为 375 或者 750,所以为了方便计算，在这里取 7.5，能够被整除方便后面的计算。

元素的 rem 值计算

```
.div{
        display: flex;
        align-items: center;
        justify-content: center;
        width: 2rem;
        height: 0.96rem;
        background:#aaa;
        font-size: 0.24rem;
        
    }
```

假定一个 div 的宽度为 100px,那么在计算时，根据设置的 1rem = 50px，所以 100px=2rem，其他所有的属性值都可以类似这样计算。font-size 一般情况下还是使用 px,因为在 rem 中，只要使用了 rem 单位都会被转换，那么在转换的时候，会出现不能被整除或者出现小数的情况比如 10.333333px，那么在显示时就可能出现一些偏差。而且如果期望在小屏幕下面显示跟大屏幕同等量的字体，但是由于 rem 的等比缩放，在小屏幕下就会存在小屏幕字体更小的情况，所以对于字体的适配更好的做法就是使用 px 和媒体查询来进行适配。

优点：

- rem 单位是根据根元素 font-size 决定大小，只要改变 font-size 的值，以 rem 为固定单位的元素大小也会发生响应式的改变

缺点：

- 必须通过一段 js 代码控制 font-size 的大小
- 控制 font-size 的 js 代码必须放在在页面第一次加载完成之前，并且放在引入的 css 样式代码之前。

(4)vw、vh 方案 css3 中引入与视口有关的新的单位 vw 和 vh，vw 表示相对于视口的宽度，vh 表示相对于视口高度

| 单位 | 含义 |
| --- | --- |
| vw | 相对于视口的宽度，视口宽度是 100vw |
| vh | 相对于视口的高度，视口宽度是 100vh |
| vmin | vw 和 vh 中较小的值 |
| vmax | vw 和 vh 中较大的值 |

vw 单位换算： 视口宽度为 100vw 占满整个视口区域，那么 1vw 相当于占整个视口宽度的 1%，所以 1px= 1/375\*100 vw 所有的页面元素都可以直接进行计算换算成 vw 单位，但是这样计算和百分比方案计算比较类似，都会比较麻烦。

但是有一个比较厉害的插件—— postcss-px-to-viewport，可以预处理 css,将 px 单位转换为 vw 单位，但是需要进行一些相关的 webpack 配置

```
{
    loader: 'postcss-loader',
    options: {
    plugins: ()=>[
        require('autoprefixer')({
        browsers: ['last 5 versions']
        }),
        require('postcss-px-to-viewport')({
        viewportWidth: 375,
        viewportHeight: 1334,
        unitPrecision: 3,
        viewportUnit: 'vw',
        selectorBlackList: ['.ignore', '.hairlines'],
                minPixelValue: 1,
                mediaQuery: false
        })
    ]
}
```

接下来是对 postcss-px-to-viewport 配置中的属性的介绍：

| 单位 | 含义 |
| --- | --- |
| viewportWidth | 视口宽度（数字） |
| viewportHeight | 视口高度（数字） |
| unitPrecision | 设置的保留小数位数（数字） |
| viewportUnit | 设置要转换的单位（字符串） |
| selectorBlackList | 不需要进行转换的类名（数组） |
| minPixelValue | 设置要替换的最小像素值（数字） |
| mediaQuery | 允许在媒体查询中转换 px（true/false） |

优点：

- 指定 vw\\vh 相对与视口的宽高，由 px 换算单位成 vw 单位比较简单
- 通过 postcss-px-to-viewport 插件进行单位转换比较方便

缺点：

- 直接进行单位换算时百分比可能出现小数,计算不方便

- 兼容性 - 大多数浏览器都支持、ie11 不支持 少数低版本手机系统 ios8、android4.4 以下不支持

最后，有一个问题，既然百分比和 vw 都是需要计算百分比，那么两个方案的不同之处？

| 单位 | 含义 |
| --- | --- |
| % | width、height 等大部分相对于直接父元素、border-radius、translate、background-size 等相对于自身 |
| vw | 只相对于视口宽度 |

以上，如有错漏，欢迎指正！

> @Author：lj0214

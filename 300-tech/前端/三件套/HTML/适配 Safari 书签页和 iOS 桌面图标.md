---
title: 适配 Safari 书签页和 iOS 桌面图标
date: 2023-11-10T09:11:14+08:00
updated: 2024-08-21T10:32:48+08:00
dg-publish: false
---

```html
<link rel="apple-touch-icon" href="/apple-touch-icon.png">  
<meta name="apple-mobile-web-app-title" content="张洪Heo">  
<link rel="bookmark" href="/apple-touch-icon.png">  
<link rel="apple-touch-icon-precomposed" sizes="180x180" href="/apple-touch-icon.png" >
```

<https://developer.apple.com/library/archive/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html>

```html
- <link rel="apple-touch-startup-image" media="screen and (device-width:320px) and (device-height:568px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_1136x640.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:320px) and (device-height:568px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_640x1136.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:414px) and (device-height:896px) and (-webkit-device-pixel-ratio:3) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2688x1242.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:414px) and (device-height:896px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_1792x828.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:375px) and (device-height:812px) and (-webkit-device-pixel-ratio:3) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_1125x2436.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:414px) and (device-height:896px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_828x1792.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:375px) and (device-height:812px) and (-webkit-device-pixel-ratio:3) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2436x1125.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:414px) and (device-height:736px) and (-webkit-device-pixel-ratio:3) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_1242x2208.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:414px) and (device-height:736px) and (-webkit-device-pixel-ratio:3) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2208x1242.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:375px) and (device-height:667px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_1334x750.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:375px) and (device-height:667px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_750x1334.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:1024px) and (device-height:1366px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2732x2048.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:1024px) and (device-height:1366px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_2048x2732.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:834px) and (device-height:1194px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2388x1668.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:834px) and (device-height:1194px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_1668x2388.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:834px) and (device-height:1112px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2224x1668.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:414px) and (device-height:896px) and (-webkit-device-pixel-ratio:3) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_1242x2688.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:834px) and (device-height:1112px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_1668x2224.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:768px) and (device-height:1024px) and (-webkit-device-pixel-ratio:2) and (orientation:portrait)" href="/img/siteicon/splashIcons/icon_1536x2048.png"/>
    - <link rel="apple-touch-startup-image" media="screen and (device-width:768px) and (device-height:1024px) and (-webkit-device-pixel-ratio:2) and (orientation:landscape)" href="/img/siteicon/splashIcons/icon_2048x1536.png"/>
```

添加到桌面

将下面的 html 代码添加到 head 标签：

```html
<meta name="apple-mobile-web-app-capable" content="yes"/>  
<meta name="apple-touch-fullscreen" content="yes"/>  
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent"/>  
<link rel="manifest" href="/manifest.json"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2048x2732.png" media="(device-width: 1024px) and (device-height: 1366px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2732x2048.png" media="(device-width: 1024px) and (device-height: 1366px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1668x2388.png" media="(device-width: 834px) and (device-height: 1194px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2388x1668.png" media="(device-width: 834px) and (device-height: 1194px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1536x2048.png" media="(device-width: 768px) and (device-height: 1024px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2048x1536.png" media="(device-width: 768px) and (device-height: 1024px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1668x2224.png" media="(device-width: 834px) and (device-height: 1112px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2224x1668.png" media="(device-width: 834px) and (device-height: 1112px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1620x2160.png" media="(device-width: 810px) and (device-height: 1080px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2160x1620.png" media="(device-width: 810px) and (device-height: 1080px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1290x2796.png" media="(device-width: 430px) and (device-height: 932px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2796x1290.png" media="(device-width: 430px) and (device-height: 932px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1179x2556.png" media="(device-width: 393px) and (device-height: 852px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2556x1179.png" media="(device-width: 393px) and (device-height: 852px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1248x2778.png" media="(device-width: 428px) and (device-height: 926px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2778x1248.png" media="(device-width: 428px) and (device-height: 926px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1170x2532.png" media="(device-width: 390px) and (device-height: 844px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2532x1170.png" media="(device-width: 390px) and (device-height: 844px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1125x2436.png" media="(device-width: 375px) and (device-height: 812px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2436x1125.png" media="(device-width: 375px) and (device-height: 812px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1242x2688.png" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2688x1242.png" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-828x1792.png" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1792x828.png" media="(device-width: 414px) and (device-height: 896px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1242x2208.png" media="(device-width: 414px) and (device-height: 736px) and (-webkit-device-pixel-ratio: 3) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-2208x1242.png" media="(device-width: 414px) and (device-height: 736px) and (-webkit-device-pixel-ratio: 3) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-750x1334.png" media="(device-width: 375px) and (device-height: 667px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1334x750.png" media="(device-width: 375px) and (device-height: 667px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-640x1136.png" media="(device-width: 320px) and (device-height: 568px) and (-webkit-device-pixel-ratio: 2) and (orientation: portrait)"/>  
<link rel="apple-touch-startup-image" href="/img/pwa/splash-1136x640.png" media="(device-width: 320px) and (device-height: 568px) and (-webkit-device-pixel-ratio: 2) and (orientation: landscape)"/>
```

添加 manifest，需要修改一下 name、short_name 为你的网站名称

```json
{  
  "name": "张洪Heo",  
  "short_name": "张洪Heo",  
  "theme_color": "#ffffff",  
  "background_color": "#ffffff",  
  "display": "standalone",  
  "scope": "/",  
  "start_url": "/",  
  "icons": [  
      {  
        "src": "img/pwa/36.png",  
        "sizes": "36x36",  
        "type": "image/png"  
      },  
      {  
          "src": "img/pwa/48.png",  
        "sizes": "48x48",  
        "type": "image/png"  
      },  
      {  
        "src": "img/pwa/72.png",  
        "sizes": "72x72",  
        "type": "image/png"  
      },  
      {  
        "src": "img/pwa/96.png",  
        "sizes": "96x96",  
        "type": "image/png"  
      },  
      {  
        "src": "img/pwa/144.png",  
        "sizes": "144x144",  
        "type": "image/png"  
      },  
      {  
        "src": "img/pwa/192.png",  
        "sizes": "192x192",  
        "type": "image/png"  
      },  
      {  
        "src": "img/pwa/196.png",  
        "sizes": "196x196",  
        "type": "image/png",  
        "purpose": "any maskable"  
      },  
      {  
        "src": "img/pwa/512.png",  
        "sizes": "512x512",  
        "type": "image/png"  
      }  
    ]  
}
```

在 张洪 Heo 公众号回复 webapp 获取设计文件素材

点击「标注」点击「导出所有切图」

放入根目录的 `/img/pwa/` 文件夹中

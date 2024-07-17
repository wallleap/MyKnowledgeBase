---
title: 005-ArkTS 动画
date: 2024-06-19 15:13
updated: 2024-06-19 15:13
---

## 属性动画

通过设置组件的 animation 属性来给组件添加动画，当组件的 width、height、opacity、backgroundColor、scale、rotate、translate 等属性变化时（放在这些属性的最后），可以实现过渡效果

```ts
.animation({
  duration: 1000,
  curve: Curve.EaseInOut
})
```

动画参数

![](https://cdn.wallleap.cn/img/pic/illustration/202406191532362.png)

## 显示动画

显示动画是通过全局 animateTo 函数来修改组件属性，实现属性变化时的过渡效果

```ts
animateTo(
  {duration: 1000}, // 动画参数
  () => {} // 溴组件属性关联的状态变量
)
```

例如

```ts
@Entry
@Component
struct Animation {
  @State left: number = 0
  @State top: number = 0
  @State text: string = '→'

  build() {
    Column() {
      Text(this.text)
        .fontSize(50)
        .fontWeight(FontWeight.Bold)
        .position({
          x: this.left,
          y: this.top
        })
      Column() {
        Button('上')
          .onClick(() => {
            animateTo({duration: 300}, () => {
              this.top -= 20
              this.text = '↑'
            })
          })
        Row() {
          Button('左')
            .margin({right: 30})
            .onClick(() => {
              animateTo({duration: 300}, () => {
                this.left -= 20
                this.text = '←'
              })
            })
          Button('右')
            .onClick(() => {
              animateTo({duration: 300}, () => {
                this.left += 20
                this.text = '→'
              })
            })
        }
        Button('下')
          .onClick(() => {
            animateTo({duration: 300}, () => {
              this.top += 20
              this.text = '↓'
            })
          })
      }
        .position({
          x: '50%',
          y: '80%'
        })
    }
    .width('100%')
  }
}
```

## 组件转场动画

在组件插入或移除时的过渡动画，通过组件的 transition 属性来配置

```ts
.transition({
  opcity: 0,
  scale: {x: 0, y: 0}
})
```

![](https://cdn.wallleap.cn/img/pic/illustration/202406191557184.png)

显式调用 animateTo 函数触发动画

```ts
if(this.isShow) {
  Text('hi')
    .transition({opcity:0}) // type: TransitionType.Insert
}
animateTo(
  {duration: 1000},
  () => {
    this.isShow = false
  }
)
```

完善之前的游戏

```ts
import curves from '@ohos.curves'
import { Header } from '../components/Common'
@Entry
@Component
struct Animation {
  @State left: number = 60
  @State top: number = 120
  @State text: string = '→'
  @State angle: number = 0
  // 摇杆
  private centerX: number = 120
  private centerY: number = 120
  private maxRadius: number = 100
  private radius: number = 20
  @State positionX: number = this.centerX
  @State positionY: number = this.centerY
  sin: number = 0
  cos: number = 0
  speed: number = 0
  taskId: number = -1

  build() {
    Column() {
      Text(this.text)
        .fontSize(50)
        .fontWeight(FontWeight.Bold)
        .position({
          x: this.left,
          y: this.top
        })
      .rotate({angle: this.angle})
      Row() {
        Circle({width: this.maxRadius * 2, height: this.maxRadius * 2})
          .fill('#17000000')
          .position({x: this.centerX - this.maxRadius, y: this.centerY - this.maxRadius})
        Circle({width: this.radius * 2, height: this.radius * 2})
          .fill('#c09c9c9c')
          .position({x: this.positionX - this.radius, y: this.positionY - this.radius})
      }
        .width(240)
        .height(240)
        .position({x: 0, y: 120})
        .onTouch(this.onTouchFn.bind(this))
    }
    .width('100%')
  }

  onTouchFn(e: TouchEvent) {
    switch (e.type) {
      case TouchType.Up:
        animateTo({curve: curves.springMotion()}, () => {
          this.positionX = this.centerX
          this.positionY = this.centerY
        })
        this.speed = 0
        clearInterval(this.taskId)
        break;
      case TouchType.Down:
        this.taskId = setInterval(() => {
          this.left += this.speed * this.cos
          this.top += this.speed * this.sin
        }, 40)
        break;
      case TouchType.Move:
        const x = e.touches[0].x
        const y = e.touches[0].y
        const vx = x - this.centerX
        const vy = y - this.centerY
        const distance = this.getDistance(vx, vy)
        let angle = Math.atan2(vy, vx)
        this.sin = Math.sin(angle)
        this.cos = Math.cos(angle)
        this.speed = 10
        animateTo({curve: curves.responsiveSpringMotion()}, () => {
          this.positionX = this.centerX + distance * this.cos
          this.positionY = this.centerY + distance * this.sin
          if(Math.abs(angle * 2) < Math.PI) {
            this.text = '→'
          } else {
            this.text = '←'
            angle = angle < 0 ? angle + Math.PI : angle - Math.PI
          }
          this.angle = angle * 180 / Math.PI
        })
        break;
      default:
        break;
    }
  }
  getDistance(x: number, y: number) {
    const d = Math.sqrt(x * x + y * y)
    return Math.min(d, this.maxRadius)
  }
}
```

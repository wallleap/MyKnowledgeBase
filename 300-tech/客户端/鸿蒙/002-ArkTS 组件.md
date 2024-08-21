---
title: 002-ArkTS 组件
date: 2024-06-17T10:46:00+08:00
updated: 2024-08-21T10:32:39+08:00
dg-publish: false
---

## 图片 Image

```ts
Image($r('app.media.icon'))  
  .width(200)  
  .borderRadius(8)  
  .interpolation(ImageInterpolation.High) // 图片特有属性，图片插值，作用是低分变率图片可以看起来清晰一点
```

声明 Image 图片显示组件并设置图片源

`Image(src: string|PixelMap|Resource)`

string 格式，通常用来加载网络图片，需要申请网络访问权限 `Image('https://xxx.png')`

```json
// src/main/module.json5
{  
  "module": {  
    "requestPermissions": [  
      {  
        "name": "ohos.permission.INTERNET"  
      }  
    ]
  }
}
```

PixelMap 格式，可以加载像素图 `Image(pixelMapObject)`

Resource 格式，加载**本地图片** `Image($r('app.media.icon))`（在 src/main/resources/base/media/icon.png）`Image($rawfile('icon.png'))`（在 src/main/resources/rawfile/icon.png）

## 文本 Text

```ts
Text('图片宽度')  
  .fontColor('#333')  
  .fontSize(26)  
Text($r('app.string.width_label'))
```

`Text(content?: string|Resource)`

string 格式直接写文本内容

`Text('你好啊')`

Resource 格式，读取本地资源文件

`Text($r('app.string.width_label'))`

在限定词目录和 `base` 目录添加上，之后切换语言就会改变

```json
// src/main/resources/zh_CN/element/string.json
{  
  "string": [  
    {  
      "name": "width_label",  
      "value": "图片宽度"  
    }  
  ]  
}
// 对应 en_US 中 value 改成 "image width"
```

## 文本输入框 TextInput

```ts
TextInput()  
  .width(150)  
  .height(30)  
  .type(InputType.Password)  // 枚举类型
  .onChange(value => this.msg = value)
```

`TextInput({placeholder: '请输入帐号或手机号', text: '111'}` 提示文本和输入文本

## 按钮 Button

```ts
Button('确定')
  .type(ButtonType.Normal)
  .onClick(e = {})
Button() {
  Image($r('app.media.search')).width(20).margin(10)
}
```

## 滑动条 Slider

```ts
Slider({  
  min: 10,  
  max: 300,  
  value: this.imageWidth,  
  step: 10,  
  style: SliderStyle.InSet,  
  direction: Axis.Horizontal,  
  reverse: false  
}).width('100%')  
  .showTips(true)  
  .showSteps(true)  
  .blockColor('#ffffffff')  
  .trackColor('#ffffe7ff')  
  .selectedColor('#ffa70ca7')  
  .trackThickness(26)  
  .onChange(value => {  
    this.imageWidth = value  
  })
```

## 布局组件 Column 和 Row

```ts
Column({space: 20}) {  
  Row() {  
    Image($r('app.media.icon'))  
      .width(this.imageWidth)  
      .borderRadius(8)  
      .interpolation(ImageInterpolation.High)  
  }.width('100%')  
    .height(350)  
    .justifyContent(FlexAlign.Center)  
  Row() {
    Text($r('app.string.width_label'))
      .fontSize(20)
      .margin({right: 8})
    TextInput({
      text: this.imageWidth.toFixed(0)
    }).layoutWeight(1)
      .height(40)
      .type(InputType.Number)
      .onChange(value => this.imageWidth = parseInt(value))
  }.width('100%')
  Row({space: 30}) {  
    Button('缩小')  
      .padding(10)  
      .width(100)  
      .backgroundColor('#ffa107a1')  
      .onClick(() => {  
        if (this.imageWidth >= 10)  
          this.imageWidth -= 10  
      })  
    Button('放大')  
      .padding(10)  
      .width(100)  
      .backgroundColor('#ffa107a1')  
      .onClick(() => {  
        if (this.imageWidth < 300)  
          this.imageWidth += 10  
      })  
  }  
  Slider({  
    min: 10,  
    max: 300,  
    value: this.imageWidth,  
    step: 10,  
    style: SliderStyle.InSet,  
    direction: Axis.Horizontal,  
    reverse: false  
  }).width('100%')  
    .showTips(true)  
    .showSteps(true)  
    .blockColor('#ffffffff')  
    .trackColor('#ffffe7ff')  
    .selectedColor('#ffa70ca7')  
    .trackThickness(26)  
    .onChange(value => {  
      this.imageWidth = value  
    })  
}.width('100%')  
  .padding(16)  
```

| 布局方式                   | 纵向布局               | 横向布局             |
| ---------------------- | ------------------ | ---------------- |
| 组件                     | 使用 `Column`        | 使用 `Row`         |
| 主轴                     | 从上往下               | 从左往右             |
| 主轴对齐<br>justifyContent | FlexAlign 枚举       | FlexAlign 枚举     |
| 交叉轴对齐<br>alignItems    | HorizontalAlign 枚举 | VerticalAlign 枚举 |

`Row({space: 20}).padding(16).magin({top: 30})`

`layoutWeight` 是布局权重，如果全都设置相同就平分，可以用于占用剩余空间

## 列表 List

List 是复杂容器，里面是 ListItem 列表项（只能有一个根组件），超过屏幕后可以滚动（正常情况不行），可以横向也可以纵向

```ts
class Item {  
  name: string  
  image: ResourceStr  
  price: number  
  discount?: number  
  
  constructor(name: string, image: ResourceStr, price: number, discount?: number) {  
    this.name = name  
    this.image = image  
    this.price = price  
    if (discount)  
      this.discount = discount  
  }  
}  
  
  
@Entry  
@Component  
struct Goods {  
  private items: Array<Item> = [  
    new Item('华为 Mate 60', $r('app.media.icon'), 6999, 500),  
    new Item('华为 Mate 50', $r('app.media.icon'), 4999),  
    new Item('Mate Book Pro X', $r('app.media.icon'), 13999, 300),  
    new Item('WatchGT 4', $r('app.media.icon'), 1438),  
    new Item('Free Buds Pro 3', $r('app.media.icon'), 1499),  
    new Item('Mate X5', $r('app.media.icon'), 12999),  
  ]  
  
  build() {  
    Column({space: 20}) {  
      Text('商品列表')  
        .fontSize(32)  
        .fontWeight(FontWeight.Bold)  
      List({space: 20}) {  
        ForEach(  
          this.items,  
          (item: Item) => {  
            ListItem() {  
              Row() {  
                Image(item.image)  
                  .width(100)  
                  .margin({right: 8})  
                Column({space: 4}) {  
                  Text(item.name)  
                    .fontSize(22)  
                    .fontWeight(FontWeight.Bold)  
                  if (item.discount) {  
                    Text('原价：￥' + item.price.toFixed(2))  
                      .fontColor('#ccc')  
                      .fontSize(12)  
                      .decoration({type: TextDecorationType.LineThrough})  
                    Text('折扣价：￥' + (item.price - item.discount).toFixed(2))  
                      .fontColor('#ffdd1616')  
                  } else {  
                    Text('￥' + item.price.toFixed(2))  
                      .fontColor('#ffdd1616')  
                  }  
                }.alignItems(HorizontalAlign.Start)  
              }  
                .backgroundColor('#fff')  
                .width('100%')  
                .padding(16)  
                .borderRadius(8)  
            }  
          }        )  
      }.listDirection(Axis.Vertical)  
       .layoutWeight(1)
    }.width('100%')  
      .backgroundColor('#ffeaeaea')  
      .padding(16)  
      .alignItems(HorizontalAlign.Start)  
  }  
}
```

ForEach 进行循环渲染，if-else 判断

```ts
ForEach(
  arr: any[],
  itemGenerator: (item: any, index?: number) => void,
  keyGenerator?: (item: any, index?: number) => string // 第三个参数是生成 key 的，可以这样 item => JSON.stringify(item)
)
LazyForEach(
  dataSource: IDataSource,
  itemGenerator: (item: any) => void,
  keyGenerator?: (item: any) => string
)
```

## 自定义组件

```ts
class Item {
  name: string
  image: ResourceStr
  price: number
  discount?: number

  constructor(name: string, image: ResourceStr, price: number, discount?: number) {
    this.name = name
    this.image = image
    this.price = price
    if (discount)
      this.discount = discount
  }
}

import {Header} from '../components/Common' // 导入使用

/* 全局公共样式函数（公共属性），声明在组件内的时候去掉 function，使用和全局是相同的 */
@Styles function fillScreen() {
  .height('100%')
  .width('100%')
  .backgroundColor('#ffeaeaea')
  .padding(16)
}

/* 继承模式（组件特有属性或方法），这个必须写在全局 */
@Extend(Text) function priceText() {
  .fontColor('#ffdd1616')
  .fontSize(16)
}

/* 全局自定义构建函数 */
@Builder function GoodItem(item) {
  Row() {
    Image(item.image)
      .width(100)
      .margin({right: 8})
    Column({space: 4}) {
      Text(item.name)
        .fontSize(22)
        .fontWeight(FontWeight.Bold)
      if (item.discount) {
        Text('原价：￥' + item.price.toFixed(2))
          .fontColor('#ccc')
          .fontSize(12)
          .decoration({type: TextDecorationType.LineThrough})
        Text('折扣价：￥' + (item.price - item.discount).toFixed(2))
          .priceText()
      } else {
        Text('优惠价：￥' + item.price.toFixed(2))
          .priceText()
      }
    }.alignItems(HorizontalAlign.Start)
  }
  .backgroundColor('#fff')
  .width('100%')
  .padding(16)
  .borderRadius(8)
}

@Entry
@Component
struct Goods {
  private items: Array<Item> = [
    new Item('华为 Mate 60', $r('app.media.icon'), 6999, 500),
    new Item('华为 Mate 50', $r('app.media.icon'), 4999),
    new Item('Mate Book Pro X', $r('app.media.icon'), 13999, 300),
    new Item('WatchGT 4', $r('app.media.icon'), 1438),
    new Item('Free Buds Pro 3', $r('app.media.icon'), 1499),
    new Item('Mate X5', $r('app.media.icon'), 12999),
  ]

  build() {
    Column({space: 20}) {
      Header({title: '商品列表'})
      List({space: 20}) {
        ForEach(
          this.items,
          (item: Item) => {
            ListItem() {
              GoodItem(item)
            }
          }
        )
        ListItem() {
          this.Footer()
        }
      }.listDirection(Axis.Vertical)
        .layoutWeight(1)
    }.fillScreen()
  }

  /* 局部构建函数，声明的时候不加 function，使用的时候需要加 this */
  @Builder Footer() {
    Text('假装有很多内容')
  }
}
```

另一个文件

```ts
/* 自定义组件，其他页面可以导入使用 */
@Component  
export struct Header {  // 导出
  private title: ResourceStr  
  build() {  
    Row() {  
      Image($r('app.media.back'))  
        .width(30)  
        .margin({right: 8})  
      Text(this.title)  
        .fontSize(32)  
        .fontWeight(FontWeight.Bold)  
      Blank()  
      Image($r('app.media.refresh'))  
        .width(30)  
    }.width('100%')  
  }  
}
```

`Blank()` 组件会占用剩余空间

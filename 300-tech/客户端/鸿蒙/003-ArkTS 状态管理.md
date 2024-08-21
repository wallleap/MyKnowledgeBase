---
title: 003-ArkTS 状态管理
date: 2024-06-18T01:24:00+08:00
updated: 2024-08-21T10:32:39+08:00
dg-publish: false
---

在声明式 UI 中，是以状态驱动视图更新（State 渲染 View，View 互动事件 State）

- 状态 State：驱动视图更新的数据（被装饰器标记的变量）
- 视图 View：基于 UI 描述渲染得到的用户界面

## State 装饰器

定义的普通变量，修改并不能引起视图的改变

可以在定义的时候使用 `@State` 装饰器标记变量

- 该变量必须初始化并且不能为空值
- 支持 Object、class、string、number、boolean、enum 类型以及这些类型的数组（数组 push、split，修改的话需要重新赋值）
- 嵌套类型（`person.girlfriend.age++`）以及数组中的对象属性不能触发视图更新

## Prop 和 Link 装饰器

当父子组件之间需要数据同步时，可以使用 `@Prop` 和 `@Link` 装饰器

例如下面是一个任务列表基本实现

```ts
class Task {
  static id: number = 1
  name: string = `任务 ${Task.id++}`
  finished: boolean = false
}

@Styles function card() {
  .width('90%')
  .padding(20)
  .backgroundColor(Color.White)
  .borderRadius(16)
  .shadow({radius: 6, color: 'rgba(0,0,0,0.01)', offsetX: 2, offsetY: 4})
}

@Extend(Text) function finishedTask() {
  .decoration({type: TextDecorationType.LineThrough})
  .fontColor('#b1b2b1')
}

@Entry
@Component
struct Todos {
  @State totalTask: number = 0
  @State finishTask: number = 0
  @State taskList: Task[] = []

  handleTaskChange() {
    this.totalTask = this.taskList.length
    this.finishTask = this.taskList.filter(item => item.finished).length
  }

  build() {
    Column({space: 16}) {
      Row() {
        Text('任务进度：')
          .fontSize(30)
          .fontWeight(FontWeight.Bold)
        Stack() {
          Progress({
            value: this.finishTask,
            total: this.totalTask,
            type: ProgressType.Ring
          })
            .width(100)
            .color('#e6d')
            .style({
              strokeWidth: 8
            })
          Row() {
            Text(this.finishTask.toString())
              .fontSize(24)
              .fontColor('#e6d')
            Text(' / ' + this.totalTask.toString())
              .fontSize(24)
          }
        }
      }
        .card()
        .margin({top: 20})
        .justifyContent(FlexAlign.SpaceAround)
      Row() {
        Button('新增任务')
          .width(100)
          .backgroundColor('#e6d')
          .onClick(() => {
            this.taskList.push(new Task())
            this.handleTaskChange()
          })
      }
        .width('90%')
        .justifyContent(FlexAlign.End)
      List({space: 10}) {
        ForEach(
          this.taskList,
          (item: Task, index) => {
            ListItem() {
              Row() {
                if(item.finished) {
                  Text(item.name)
                    .finishedTask()
                } else {
                  Text(item.name)
                }
                Checkbox()
                  .selectedColor('#e6d')
                  .select(item.finished)
                  .onChange((val) => {
                    item.finished = val
                    this.handleTaskChange()
                  })
              }
                .width('100%')
                .justifyContent(FlexAlign.SpaceBetween)
            }
              .card()
              .swipeAction({end: this.DeleteButton(index)})
          }
        )
      }
        .layoutWeight(1)
        .alignListItem(ListItemAlign.Center)
    }
      .width('100%')
      .height('100%')
      .backgroundColor('#f1f2f3')
  }

  @Builder DeleteButton(index: number) {
    Button() {
      Image($r('app.media.delete'))
        .width(20)
        .fillColor('#fff')
    }
      .margin({left: 10})
      .width(40)
      .height(40)
      .type(ButtonType.Circle)
      .backgroundColor(Color.Red)
      .onClick(() => {
        this.taskList.splice(index, 1)
        this.handleTaskChange()
      })
  }
}
```

根据功能模块拆分成组件，上面统计的不需要双向同步就使用 `@Prop`

```ts
@Component
struct TaskStatistics {
  @Prop finishTask: number // 使用 @Prop 装饰器
  @Prop totalTask: number // 不能重新赋值

  build() {
    Row() {
      Text('任务进度：')
        .fontSize(30)
        .fontWeight(FontWeight.Bold)
      Stack() {
        Progress({
          value: this.finishTask,
          total: this.totalTask,
          type: ProgressType.Ring
        })
          .width(100)
          .color('#e6d')
          .style({
            strokeWidth: 8
          })
        Row() {
          Text(this.finishTask.toString())
            .fontSize(24)
            .fontColor('#e6d')
          Text(' / ' + this.totalTask.toString())
            .fontSize(24)
        }
      }
    }
    .card()
    .margin({top: 20})
    .justifyContent(FlexAlign.SpaceAround)
  }
}
```

下面任务列表需要修改则使用 `@Link`，这样可以双向同步

```ts
@Component
struct TaskList {
  @Link totalTask: number
  @Link finishTask: number
  @State taskList: Task[] = []

  handleTaskChange() {
    this.totalTask = this.taskList.length
    this.finishTask = this.taskList.filter(item => item.finished).length
  }

  build() {
    Column({space: 16}) {
      Row() {
        Button('新增任务')
          .width(100)
          .backgroundColor('#e6d')
          .onClick(() => {
            this.taskList.push(new Task())
            this.handleTaskChange()
          })
      }
      .width('90%')
      .justifyContent(FlexAlign.End)
      List({space: 10}) {
        ForEach(
          this.taskList,
          (item: Task, index) => {
            ListItem() {
              Row() {
                if(item.finished) {
                  Text(item.name)
                    .finishedTask()
                } else {
                  Text(item.name)
                }
                Checkbox()
                  .selectedColor('#e6d')
                  .select(item.finished)
                  .onChange((val) => {
                    item.finished = val
                    this.handleTaskChange()
                  })
              }
              .width('100%')
              .justifyContent(FlexAlign.SpaceBetween)
            }
            .card()
            .swipeAction({end: this.DeleteButton(index)})
          }
        )
      }
      .layoutWeight(1)
      .alignListItem(ListItemAlign.Center)
    }
  }

  @Builder DeleteButton(index: number) {
    Button() {
      Image($r('app.media.delete'))
        .width(20)
        .fillColor('#fff')
    }
    .margin({left: 10})
    .width(40)
    .height(40)
    .type(ButtonType.Circle)
    .backgroundColor(Color.Red)
    .onClick(() => {
      this.taskList.splice(index, 1)
      this.handleTaskChange()
    })
  }
}
```

使用的时候不一样

```ts
@Entry  
@Component  
struct Todos {  
  @State totalTask: number = 0  
  @State finishTask: number = 0  
  
  build() {  
    Column({space: 16}) {  
      TaskStatistics({finishTask: this.finishTask, totalTask: this.totalTask})  // 传的仍然是变量
      TaskList({finishTask: $finishTask, totalTask: $totalTask})  // 传的是引用
    }  
      .width('100%')  
      .height('100%')  
      .backgroundColor('#f1f2f3')  
  }  
}
```

- `@Prop` 装饰器修饰的变量需要从父组件中传递过来，并且它是重新生成的一个拷贝的新的变量，修改不会引起父组件中数据改变
- `@Link` 装饰器也需要从父组件中传递 `@State` 修饰的变量，传递的是引用，修改会同步

| 区别            | `@Prop`                                                           | `@Link`                                                                                       |
| ------------- | ----------------------------------------------------------------- | --------------------------------------------------------------------------------------------- |
| **同步类型**      | 单向同步                                                              | 双向同步                                                                                          |
| **允许装饰的变量类型** | 只支持 string、number、boolean、enum 类型；父组件对象类型，子组件是**对象属性**；不可以是数组、any | 父子类型一致：string、number、boolean、enum、object、class，以及他们的数组；数组中元素增、删、替换会引起刷新嵌套类型以及数组中的对象属性无法触发视图更新 |
| **初始化方式**     | 不允许子组件初始化                                                         | 父组件传递，禁止子组件初始化                                                                                |

```ts
class StateInfo {
  totalTask: number = 0
  finishTask: number = 0
}

// 父
@State stat: StatInfo = new StatInfo()
...
TaskStatistics({finishTask: this.stat.finishTask, totalTask: this.stat.totalTask})
TaskList(stat: $stat)

// 子 使用 this.totalTask this.finishTask
@Prop totalTask: number
@Prop finishTask: number

// 子 使用 this.stat.finishTask this.stat.totalTask
@Link stat: StatInfo
```

## Provide 和 Consume 装饰器

`@Provide` 和 `@Consume` 可以跨组件提供类似于 `@State` 和 `@Link` 的双向同步

不需要手动传，内部维护

```ts
class StateInfo {
  totalTask: number = 0
  finishTask: number = 0
}

// 父
@Provide stat: StatInfo = new StatInfo()
...
TaskStatistics() // 不用传
TaskList()

// 子 使用 this.stat.finishTask this.stat.totalTask
@Consume stat: StatInfo
```

## ObjectLink 和 Observed 装饰器

`@ObjectLink` 和 `@Observed` 装饰器用于在涉及**嵌套对象**或**数组元素为对象**的场景中进行双向数据同步

```ts
@Observed // 对象上边添加这个
class Person {
  name: string
  age: number
  child: Person

  constructor(name: string, age: number, child?: Person) {
    this.name = name
    this.age = age
    this.child = child
  }
}

@Entry
@Component
struct Parent {
  @State p: Person = new Person('Jack', 26, new Person('Tom', 5))
  @State childs: Person[] = [
    new Person('Tommy', 2),
    new Person('Lucy', 3)
  ]

  build() {
    Column({space: 20}) {
      Child({p: this.p.child})
        .onClick(() => this.p.child.age++)
      Text('Childs:')
      ForEach(
        this.childs,
        p => {
          Child({p: p}).onClick(() => p.age++)
        }
      )
    }.width('100%')
      .padding(16)
  }
}

@Component
struct Child {
  @ObjectLink p: Person // 抽成组件，通过传递参数

  build() {
    Column() {
      Text(`${this.p.name}, ${this.p.age} age old.`)
    }
  }
}
```

所以现在可以完成之前的任务列表了

```ts
@Observed
class Task {
  static id: number = 1
  name: string = `任务 ${Task.id++}`
  finished: boolean = false
}

@Styles function card() {
  .width('90%')
  .padding(20)
  .backgroundColor(Color.White)
  .borderRadius(16)
  .shadow({radius: 6, color: 'rgba(0,0,0,0.01)', offsetX: 2, offsetY: 4})
}

@Extend(Text) function finishedTask() {
  .decoration({type: TextDecorationType.LineThrough})
  .fontColor('#b1b2b1')
}

@Entry
@Component
struct Todos {
  @State totalTask: number = 0
  @State finishTask: number = 0

  build() {
    Column({space: 16}) {
      TaskStatistics({finishTask: this.finishTask, totalTask: this.totalTask})
      TaskList({finishTask: $finishTask, totalTask: $totalTask})
    }
      .width('100%')
      .height('100%')
      .backgroundColor('#f1f2f3')
  }
}

@Component
struct TaskStatistics {
  @Prop finishTask: number
  @Prop totalTask: number

  build() {
    Row() {
      Text('任务进度：')
        .fontSize(30)
        .fontWeight(FontWeight.Bold)
      Stack() {
        Progress({
          value: this.finishTask,
          total: this.totalTask,
          type: ProgressType.Ring
        })
          .width(100)
          .color('#e6d')
          .style({
            strokeWidth: 8
          })
        Row() {
          Text(this.finishTask.toString())
            .fontSize(24)
            .fontColor('#e6d')
          Text(' / ' + this.totalTask.toString())
            .fontSize(24)
        }
      }
    }
    .card()
    .margin({top: 20})
    .justifyContent(FlexAlign.SpaceAround)
  }
}

@Component
struct TaskList {
  @Link totalTask: number
  @Link finishTask: number
  @State taskList: Task[] = []

  handleTaskChange() {
    this.totalTask = this.taskList.length
    this.finishTask = this.taskList.filter(item => item.finished).length
  }

  build() {
    Column({space: 16}) {
      Row() {
        Button('新增任务')
          .width(100)
          .backgroundColor('#e6d')
          .onClick(() => {
            this.taskList.push(new Task())
            this.handleTaskChange()
          })
      }
      .width('90%')
      .justifyContent(FlexAlign.End)
      List({space: 10}) {
        ForEach(
          this.taskList,
          (item: Task, index) => {
            ListItem() {
              TaskItem({item: item, onTaskChange: this.handleTaskChange.bind(this)})  // 通过 bind 绑定一下 this
            }
            .card()
            .swipeAction({end: this.DeleteButton(index)})
          }
        )
      }
      .layoutWeight(1)
      .alignListItem(ListItemAlign.Center)
    }
  }

  @Builder DeleteButton(index: number) {
    Button() {
      Image($r('app.media.delete'))
        .width(20)
        .fillColor('#fff')
    }
    .margin({left: 10})
    .width(40)
    .height(40)
    .type(ButtonType.Circle)
    .backgroundColor(Color.Red)
    .onClick(() => {
      this.taskList.splice(index, 1)
      this.handleTaskChange()
    })
  }
}

@Component
struct TaskItem {
  @ObjectLink item: Task
  onTaskChange: () => void // 再传递一个函数

  build() {
    Row() {
      if(this.item.finished) {
        Text(this.item.name)
          .finishedTask()
      } else {
        Text(this.item.name)
      }
      Checkbox()
        .selectedColor('#e6d')
        .select(this.item.finished)
        .onChange((val) => {
          this.item.finished = val
          this.onTaskChange()
        })
    }
    .width('100%')
    .justifyContent(FlexAlign.SpaceBetween)
  }
}

```

---
title: JS 中优雅地使用 async-await
date: 2023-08-09 10:12
updated: 2023-08-09 10:14
cover: //cdn.wallleap.cn/img/post/1.jpg
image-auto-upload: true
author: Luwang
comments: true
aliases:
  - JS 中优雅地使用 async-await
rating: 1
tags:
  - JS
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

小知识，大挑战！本文正在参与「[程序员必备小知识](https://juejin.cn/post/7008476801634680869 "https://juejin.cn/post/7008476801634680869")」创作活动

本文已参与「[掘力星计划](https://juejin.cn/post/7012210233804079141/ "https://juejin.cn/post/7012210233804079141/")」，赢取创作大礼包，挑战创作激励金。

## 酝酿一下感情

### jQuery 的 $.ajax

在开始之前我们先来聊聊我的 js 异步之路。在我还在学校的时候，那时候还是 `jQuery` 的天下，我直接接触到并且经常使用的异步操作就是网络请求，一手 [$.ajax](https://link.juejin.cn/?target=https%3A%2F%2Fapi.jquery.com%2FjQuery.ajax%2F "https://api.jquery.com/jQuery.ajax/") 走天下，伴我过了大二到毕业后差不多大半年的时间。

```js
$.ajax( "/xxx" )
  .done(function() {
    // success !!! do something...
  })
  .fail(function() {
    // fail !!! do something...
  })
  .always(function() {
    // loading finished..
  });
```

不可否认，`$.ajax` 这个东西还是挺好使的，在面对大部分场景只有一个请求的情况下，完全胜任甚至觉得很棒

但是有个大大的问题，那就是面对请求链的时候就会特别特别的糟心，比如一个请求依赖于另一个请求的结果，两个可能还无所谓，要是五个八个的，可能想要直接自杀。。。

```js
$.ajax('/xxx1')
  .done(function() {
    // success !!! do something...
    $.ajax('/xxx2')
      .done(function() {
        // success !!! do something...
        $.ajax('/xxx3')
          .done(function() {
            // success !!! do something...
            $.ajax('/xxx4')
              .done(function() {
                // success !!! do something...
                $.ajax('/xxx5')
                  .done(function() {
                    // success !!! do something...
                    // more...
                  })
                  .fail(function() {
                    // fail !!! do something...
                  })
                  .always(function() {
                    // loading finished..
                  });
              })
              .fail(function() {
                // fail !!! do something...
              })
              .always(function() {
                // loading finished..
              });
          })
          .fail(function() {
            // fail !!! do something...
            $.ajax('/xxx6')
              .done(function() {
                // success !!! do something...
                $.ajax('/xxx7')
                  .done(function() {
                    // success !!! do something...
                    // more....
                  })
                  .fail(function() {
                    // fail !!! do something...
                  })
                  .always(function() {
                    // loading finished..
                  });
              })
              .fail(function() {
                // fail !!! do something...
              })
              .always(function() {
                // loading finished..
              });
          })
          .always(function() {
            // loading finished..
          });
      })
      .fail(function() {
        // fail !!! do something...
      })
      .always(function() {
        // loading finished..
      });
  })
  .fail(function() {
    // fail !!! do something...
  })
  .always(function() {
    // loading finished..
  });
```

抱歉，我不知道你可以套这么多层。。。，但事实就是 TM 经常出现这样的流程，大伙儿说说，这不能怪产品吧？？？~只能怪自己学艺不精~

像这样链式操作，我觉得吧，是个人可能都是奔溃的，先不说代码的可读性，就拿天天在变化的产品需求来说，也许先前是 `请求1` 结束之后紧接着 `请求2` 、 `请求3` ，后面产品大手一挥，我觉得这个流程不大对，后面就变成了 `请求2`、 `请求3` 、 `请求1`，这尼玛套娃怎么改？可能有人会有疑问，为啥不用 `axios` 、 `await` 、`async` 呢？这个就不得不提项目代码是 08 年开写的 JSP 了。。。。在整了大半年的屎上拉屎以后，迎来了大大的转机，新写的项目开始往 `Vue` 上面转，并且放弃一部分兼容性，我 TM 直接起飞。。。

### Webpack 时代的开始

新的项目直接 Vue + Webpack，我直接就给安排上 `axios` 、 `await` 、`async` ，现在代码非常好使，嵌套 N 层的代码没了

```js
const r1 = await doSomthing1();
if (r1.xxx === 1) {
  const r2 = await doSomthing2(r1);
  const r3 = await doSomthing3(r2);
  // do something....
} else {
  const r4 = await doSomthing4(r1);
  const r5 = await doSomthing5(r4);
  // do something....
}
// do something....
```

但是上面的代码存在一个问题，如果某个任务报错，那么代码直接就终止了。。。这样不符合我们的预期啊，那我们加上 `try catch`

```js
let r1;
try {
  r1 = await doSomthing1();
} catch (e) {
  // do something...
  return;
}
if (r1) {
  if (r1.xxx === 1) {
    let r2;
    try {
      r2 = await doSomthing2(r1);
    } catch (e) {
      // do something...
      return;
    }
    if (r2) {
      let r3;
      try {
        r3 = await doSomthing3(r2);
      } catch (e) {
        // do something...
        return;
      }
      // do something...
    }
  } else {
    let r4;
    try {
      r4 = await doSomthing4(r1);
    } catch (e) {
      // do something...
      return;
    }
    if (r4) {
      let r5;
      try {
        r5 = await doSomthing5(r4);
      } catch (e) {
        // do something...
        return;
      }
    }
    // do something...
  }
  // do something...
}
```

？？？

优化了，等于没优化。。。

这时候我想聪明的小伙伴可能会说了，这是啥煎饼玩意儿。而呆滞的小伙伴已经开始想怎么解决这样的问题了。。。

### 深入了解 Promise

我们来看一下 [Promise](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise") 的定义

```js
/**
 * Represents the completion of an asynchronous operation
 */
interface Promise<T> {
    /**
     * Attaches callbacks for the resolution and/or rejection of the Promise.
     * @param onfulfilled The callback to execute when the Promise is resolved.
     * @param onrejected The callback to execute when the Promise is rejected.
     * @returns A Promise for the completion of which ever callback is executed.
     */
    then<TResult1 = T, TResult2 = never>(onfulfilled?: ((value: T) => TResult1 | PromiseLike<TResult1>) | undefined | null, onrejected?: ((reason: any) => TResult2 | PromiseLike<TResult2>) | undefined | null): Promise<TResult1 | TResult2>;

    /**
     * Attaches a callback for only the rejection of the Promise.
     * @param onrejected The callback to execute when the Promise is rejected.
     * @returns A Promise for the completion of the callback.
     */
    catch<TResult = never>(onrejected?: ((reason: any) => TResult | PromiseLike<TResult>) | undefined | null): Promise<T | TResult>;
}
```

`then` 和 `catch` 都会返回一个新的 `Promise` ，我相信很多小伙伴都已经想到了怎么解决方法，需要使用 `try catch` 是因为它会报错，那我们返回一个 `永远不会报错的结果` 不就行了？说干就干

### 消灭嵌套

```js
function any(promise) {
  return promise.then((v) => v).catch((_) => null);
}
```

这样就完全解决了啊？？？通过判断是否有值来判断是否成功，就不用再写 `try catch` 了，但是这样的代码有点不大好使，如果 `then` 返回的是一个 `void` 那么就完犊子了，一个 `undefined` 一个 `null` ，这还判断个锤子，我们再来改进一下

```js
function any(promise) {
  return promise
    .then((v) => ({ ok: v, hasErr: false }))
    .catch((e) => ({ err: e, hasErr: true }));
}
```

使用的话

```js
const r = await any(doSomething());
if (r.hasErr) {
  console.log(r.err);
  return;
}
console.log(r.ok);
```

现在看起来是不是很完美呢，赶紧和小伙伴推销一下。

小伙伴：？？？这啥煎饼玩意儿，不用不用。

我：这个我写的，在异步中用起来很好使的，告别嵌套 `try catch` ，巴拉巴拉。。。

小伙伴：好的，下次一定用。

大家肯定有遇到过这样的情况，大家写的代码互相看不起，只要不是三方库，大家都是能不用同事写的就不用。。。

### await-to-js

我都以为只有我一人欣赏，这一份优雅。事情出现转机，某天我正在刷 github，发现了一个和我差不多异曲同工之妙的东西 [await-to-js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fscopsy%2Fawait-to-js "https://github.com/scopsy/await-to-js") ，几行代码透露了和我一样的执着

```js
// 下面是最新的代码
/**
 * @param { Promise } promise
 * @param { Object= } errorExt - Additional Information you can pass to the err object
 * @return { Promise }
 */
export function to<T, U = Error> (
  promise: Promise<T>,
  errorExt?: object
): Promise<[U, undefined] | [null, T]> {
  return promise
    .then<[null, T]>((data: T) => [null, data])
    .catch<[U, undefined]>((err: U) => {
      if (errorExt) {
        Object.assign(err, errorExt);
      }

      return [err, undefined];
    });
}

export default to;
```

再贴上使用示例

```js
import to from 'await-to-js';
// If you use CommonJS (i.e NodeJS environment), it should be:
// const to = require('await-to-js').default;

async function asyncTaskWithCb(cb) {
     let err, user, savedTask, notification;

     [ err, user ] = await to(UserModel.findById(1));
     if(!user) return cb('No user found');

     [ err, savedTask ] = await to(TaskModel({userId: user.id, name: 'Demo Task'}));
     if(err) return cb('Error occurred while saving task');

    if(user.notificationsEnabled) {
       [ err ] = await to(NotificationService.sendNotification(user.id, 'Task Created'));
       if(err) return cb('Error while sending notification');
    }

    if(savedTask.assignedUser.id !== user.id) {
       [ err, notification ] = await to(NotificationService.sendNotification(savedTask.assignedUser.id, 'Task was created for you'));
       if(err) return cb('Error while sending notification');
    }

    cb(null, savedTask);
}

async function asyncFunctionWithThrow() {
  const [err, user] = await to(UserModel.findById(1));
  if (!user) throw new Error('User not found');
  
}
```

是不是感觉回来了，嵌套不再。。。

艹，为了让小伙伴用上一行的代码，我只能忍痛推荐 [await-to-js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fscopsy%2Fawait-to-js "https://github.com/scopsy/await-to-js") ，发上 github 地址，小伙伴：八百多 star (ps: 现在 2K+) 质量可靠，看了一下示例，嗯嗯，很不错，很完美，后面。。。后面的事不用我多说了，我自己写的也全换成了 [await-to-js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fscopsy%2Fawait-to-js "https://github.com/scopsy/await-to-js") 。。。

我待世界如初恋，初恋却伤我千百遍

## 总结

- 我实现的版本其实存在着一点点问题的，在 JS 这样 `灵活` 的语言中，我改了返回值，别人就能直接抄我的家，类型不够严谨，要是放 TS 里，那就只能说一点小毛病，新加了 `ok` 、 `err` 、 `hasErr` 增加了一小点点 case，但并不致命
- [await-to-js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fscopsy%2Fawait-to-js "https://github.com/scopsy/await-to-js") 中一点点的设计哲学，为啥把错误放在数组的第一个位置，而不是把成功放在第一个位置，就很明示：永远谨记错误，把错误放在第一位，而不是很 `自信` 成功，就忘记错误的惨痛。

```js
const [, result] = await to(iWillSucceed());
```

如果文章对您有帮助的话，欢迎 `点赞` 、 `评论` 、 `收藏` 、 `分享` ，您的支持是我码字的动力，万分感谢！！！🌈

## 参考资料

- [$.ajax](https://link.juejin.cn/?target=https%3A%2F%2Fapi.jquery.com%2FjQuery.ajax%2F "https://api.jquery.com/jQuery.ajax/")
- [Promise](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.mozilla.org%2Fzh-CN%2Fdocs%2FWeb%2FJavaScript%2FReference%2FGlobal_Objects%2FPromise "https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise")
- [await-to-js](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fscopsy%2Fawait-to-js "https://github.com/scopsy/await-to-js")

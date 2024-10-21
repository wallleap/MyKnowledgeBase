---
title: Flip 动画做拖拽排序效果
date: 2024-09-08T12:55:24+08:00
updated: 2024-09-08T12:57:53+08:00
dg-publish: false
---

- First 开始
- Last 结束
- Invert 翻转
- Play 播放

先来看效果

![在这里插入图片描述](https://i-blog.csdnimg.cn/blog_migrate/9cd77c8ad5221d52b44274c0aa1d86ac.gif)

index.html 文件

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
    <link rel="stylesheet" href="./index.css" />
  </head>
  <body>
    <div class="list">
      <div draggable="true" class="list-item">1</div>
      <div draggable="true" class="list-item">2</div>
      <div draggable="true" class="list-item">3</div>
      <div draggable="true" class="list-item">4</div>
      <div draggable="true" class="list-item">5</div>
    </div>
    <script src="./Flip.js"></script>
    <script src="./index.js"></script>
  </body>
</html>
```

index.css 文件

```css
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
}

.list {
  width: 500px;
}

.list-item {
  margin: 5px 0;
  padding: 0 20px;
  line-height: 40px;
  height: 40px;
  background: linear-gradient(to right, #267871, #136a8a);
  color: #fff;
  cursor: move;
  user-select: none;
  border-radius: 5px;
}
.list-item.moving {
  background: transparent;
  color: transparent;
  border: 1px dashed #ccc;
}
```

index.js 文件

```javascript
const list = document.querySelector('.list');
let sourceNode; // 当前正在拖动的是哪个元素
let flip;
list.ondragstart = (e) => {
  setTimeout(() => {
    e.target.classList.add('moving');
  }, 0);
  sourceNode = e.target;
  e.dataTransfer.effectAllowed = 'move';
  flip = new Flip(list.children, 0.3);
};
list.ondragover = (e) => {
  e.preventDefault();
};
list.ondragenter = (e) => {
  e.preventDefault();
  if (e.target === list || e.target === sourceNode) {
    return;
  }
  const children = Array.from(list.children);
  const sourceIndex = children.indexOf(sourceNode);
  const targetIndex = children.indexOf(e.target);
  if (sourceIndex < targetIndex) {
    list.insertBefore(sourceNode, e.target.nextElementSibling);
  } else {
    list.insertBefore(sourceNode, e.target);
  }
  flip.play();
};

list.ondragend = (e) => {
  e.target.classList.remove('moving');
};

```

Flip.js 文件

```javascript
const Flip = (function () {
  class FlipDom {
    constructor(dom, duration = 0.5) {
      this.dom = dom;
      this.transition =
        typeof duration === 'number' ? `${duration}s` : duration;
      this.firstPosition = {
        x: null,
        y: null,
      };
      this.isPlaying = false;
      this.transitionEndHandler = () => {
        this.isPlaying = false;
        this.recordFirst();
      };
    }

    getDomPosition() {
      const rect = this.dom.getBoundingClientRect();
      return {
        x: rect.left,
        y: rect.top,
      };
    }

    recordFirst(firstPosition) {
      if (!firstPosition) {
        firstPosition = this.getDomPosition();
      }
      this.firstPosition.x = firstPosition.x;
      this.firstPosition.y = firstPosition.y;
    }

    *play() {
      if (!this.isPlaying) {
        this.dom.style.transition = 'none';
        const lastPosition = this.getDomPosition();
        const dis = {
          x: lastPosition.x - this.firstPosition.x,
          y: lastPosition.y - this.firstPosition.y,
        };
        if (!dis.x && !dis.y) {
          return;
        }
        this.dom.style.transform = `translate(${-dis.x}px, ${-dis.y}px)`;
        yield 'moveToFirst';
        this.isPlaying = true;
      }

      this.dom.style.transition = this.transition;
      this.dom.style.transform = `none`;
      this.dom.removeEventListener('transitionend', this.transitionEndHandler);
      this.dom.addEventListener('transitionend', this.transitionEndHandler);
    }
  }

  class Flip {
    constructor(doms, duration = 0.5) {
      this.flipDoms = [...doms].map((it) => new FlipDom(it, duration));
      this.flipDoms = new Set(this.flipDoms);
      this.duration = duration;
      this.flipDoms.forEach((it) => it.recordFirst());
    }

    addDom(dom, firstPosition) {
      const flipDom = new FlipDom(dom, this.duration);
      this.flipDoms.add(flipDom);
      flipDom.recordFirst(firstPosition);
    }

    play() {
      let gs = [...this.flipDoms]
        .map((it) => {
          const generator = it.play();
          return {
            generator,
            iteratorResult: generator.next(),
          };
        })
        .filter((g) => !g.iteratorResult.done);

      while (gs.length > 0) {
        document.body.clientWidth;
        gs = gs
          .map((g) => {
            g.iteratorResult = g.generator.next();
            return g;
          })
          .filter((g) => !g.iteratorResult.done);
      }
    }
  }

  return Flip;
})();


```

---
title: cavas 获取图片前三种主要颜色并设置背景色
date: 2024-09-24T02:45:33+08:00
updated: 2024-09-24T03:08:05+08:00
dg-publish: false
---

可以使用第三方库 colorthief

```js
const colorThief = new ColorThief()

colorThief.getPalette(img, 3).then(res => {
  const [c1, c2, c3] = res.map(c => `rgb(${c[0]}, ${c[1]}, $c[2])`)
  document.documentElement.style.setProperty('--c1', c1)
  document.documentElement.style.setProperty('--c2', c2)
  document.documentElement.style.setProperty('--c3', c3)
})
```

手写简单的没有近似算法/颜色聚合算法

```js
class ColorThief {
  constructor() {
    this.canvas = document.createElement('canvas');
    this.context = this.canvas.getContext('2d');
  }

  getImageData(img) {
    this.canvas.width = img.width;
    this.canvas.height = img.height;
    this.context.drawImage(img, 0, 0, img.width, img.height);
    return this.context.getImageData(0, 0, img.width, img.height);
  }

  getColor(img) {
    return new Promise((resolve, reject) => {
      if (!(img instanceof HTMLImageElement)) {
        reject(new Error('Input is not an image element'));
        return;
      }

      if (img.complete) {
        const imageData = this.getImageData(img);
        const color = this.getDominantColor(imageData.data);
        resolve(color);
      } else {
        img.onload = () => {
          const imageData = this.getImageData(img);
          const color = this.getDominantColor(imageData.data);
          resolve(color);
        };
        img.onerror = () => reject(new Error('Image failed to load'));
      }
    });
  }

  getPalette(img, colorCount) {
    return new Promise((resolve, reject) => {
      if (!(img instanceof HTMLImageElement)) {
        reject(new Error('Input is not an image element'));
        return;
      }

      if (img.complete) {
        const imageData = this.getImageData(img);
        const palette = this.getColorPalette(imageData.data, colorCount);
        resolve(palette);
      } else {
        img.onload = () => {
          const imageData = this.getImageData(img);
          const palette = this.getColorPalette(imageData.data, colorCount);
          resolve(palette);
        };
        img.onerror = () => reject(new Error('Image failed to load'));
      }
    });
  }

  getDominantColor(pixels) {
    const colorCounts = {};
    let maxCount = 0;
    let dominantColor = null;

    for (let i = 0; i < pixels.length; i += 4) {
      const r = pixels[i];
      const g = pixels[i + 1];
      const b = pixels[i + 2];
      const rgb = `${r},${g},${b}`;

      colorCounts[rgb] = (colorCounts[rgb] || 0) + 1;

      if (colorCounts[rgb] > maxCount) {
        maxCount = colorCounts[rgb];
        dominantColor = [r, g, b];
      }
    }

    return dominantColor;
  }

  getColorPalette(pixels, colorCount) {
    const colorCounts = {};

    for (let i = 0; i < pixels.length; i += 4) {
      const r = pixels[i];
      const g = pixels[i + 1];
      const b = pixels[i + 2];
      const rgb = `${r},${g},${b}`;

      colorCounts[rgb] = (colorCounts[rgb] || 0) + 1;
    }

    return Object.entries(colorCounts)
      .sort((a, b) => b[1] - a[1])
      .slice(0, colorCount)
      .map(([color]) => color.split(',').map(Number));
  }
}

const colorThief = new ColorThief()
const img = document.querySelector('img')
const html = document.documentElement
console.log(colorThief.getImageData(img))
colorThief.getColor(img).then(res => {
  console.log(`rgb(${res})`)
})
colorThief.getPalette(img, 3).then(c => {
  console.log(c)
  const [c1, c2, c3] = c.map(color => `rgb(${color[0]}, ${color[1]}, ${color[2]})`)
  html.style.background = `linear-gradient(45deg, ${c1}, ${c2}, ${c3})`;
})
```

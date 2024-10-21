---
title: 原生音频 API 进行音频处理
date: 2024-10-14T02:11:04+08:00
updated: 2024-10-14T02:41:21+08:00
dg-publish: false
---

例如音频可视化

```html
<canvas id="cvs"></canvas>
<audio id="audio" controls src="./music.mp3"></audio>
<script>
  const audio = document.getElementById('audio')
  const canvas = document.getElementById('cvs')
  const ctx = canvas.getContext('2d')

  const initCvs = () => {
    cvs.width = window.innerWidth * devicePixelRatio
    cvs.height = window.innerHeight * 0.25 * devicePixelRatio
  }
  initCvs()

  let isInit = false
  let dataArray, analyser
  audio.onplay = () => {
    if (isInit) {
      return
    }
    const audCtx = new AudioContext()  // 音频上下文
    const source = audCtx.createMediaElementSource(audio)  // 音频源
    analyser = audCtx.createAnalyser()
    analyser.fftSize = 1024
    dataArray = new Uint8Array(analyser.frequencyBinCount)
    source.connect(analyser)
    analyser.connect(audCtx.destination)
    isInit = true
  }

  const draw = () => {
    requestAnimationFrame(draw)
    const { width, height } = canvas
    ctx.clearRect(0, 0, width, height)
    if (!isInit) return
    analyser.getByteFrequencyData(dataArray)
    const len = dataArray.length
    const barWidth = width / len
    ctx.fillStyle = 'rgb(0, 0, 255)'
    for (let i = 0; i < len; i++) {
      const value = dataArray[i]
      const barHeight = value / 255 * height
      const x = i * barWidth
      const y = height - barHeight
      ctx.fillRect(x, y, barWidth, barHeight)
    }
  }
  draw()
</script>
```

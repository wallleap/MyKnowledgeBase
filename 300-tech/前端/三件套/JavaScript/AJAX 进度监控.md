---
title: AJAX 进度监控
date: 2024-08-19T04:30:15+08:00
updated: 2024-10-15T01:36:57+08:00
dg-publish: 
---

XHR 的 progress 事件

```javascript
function ajaxProgress() {
    // 创建一个用于展示进度条的 <div>
    let progressBar = document.createElement('div');
    progressBar.id = 'progress-bar';
    document.body.appendChild(progressBar);

    function updateProgressBar() {
        const xhr = new XMLHttpRequest(); // 或使用 Fetch API 的 fetch()
        xhr.open("GET", "/api/progress"); // 请求一个示例 API 来获取进度
        xhr.onload = () => {
            if (xhr.status >= 200 && xhr.status < 300) {
                const progressValue = xhr.responseText;
                progressBar.style.width = `${progressValue}%`; // 在 HTML 页面中更新进度条的宽度以表示进度
            }
        };
        xhr.onprogress = (e) => {
            if (e.lengthComputable) {
                const percentComplete = Math.round((e.loaded / e.total) * 100);
                progressBar.style.width = `${percentComplete}%`;
            }
        };
        xhr.send(); // 发起请求以获得进度
    }

    updateProgressBar();
}
```

上传和下载

```js
xhr.upload.addEventListener('progress', e => {
  console.log(e.loaded, e.total)
})
xhr.addEventListener('progress', e => {
  console.log(e.loded, e.total)
  onProgress && onProgress({loaded: e.loaded, total: e.total})
})
xhr.upload.
```

fetch 响应进度

```js
export function request(options = {}) {
  const {url, method = 'GET', onProgress, data = null} = options
  return new Promise(async (resolve) => {
    const resp = await fetch(url, {
      method,
      body: data
    })
    const total = +resp.headers.get('content-length')
    const decoder = new TextDecoder()
    let body = ''
    const reader = resp.body.getReader()
    let loaded = 0
    while(1) {
      const {done, value} = await reader.read()
      if(done) break
      loaded += value.length
      body += decoder.decode(value)
      onProgress && onProgress({loaded, total})
    }
    resolve(body)
  })
}
```

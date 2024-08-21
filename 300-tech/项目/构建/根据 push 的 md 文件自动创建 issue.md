---
title: 根据 push 的 md 文件自动创建 issue
date: 2023-08-10T05:23:00+08:00
updated: 2024-08-21T10:32:46+08:00
dg-publish: false
aliases:
  - 根据 push 的 md 文件自动创建 issue
author: Luwang
category: web
comments: true
cover: //cdn.wallleap.cn/img/post/1.jpg
description: 文章描述
image-auto-upload: true
keywords:
  - 关键词1
  - 关键词2
  - 关键词3
rating: 1
source: #
tags:
  - blog
url: https://obsius.site/19040o4a525y0t3m712a
---

根据以下步骤操作：

1. 在仓库的 settings 中新增 Actions secrets and variables `ACCESS_TOKEN`
2. 在仓库中新建 workflow 文件
3. push md 文件

## 新增变量

前往 <https://github.com/settings/tokens> 创建 classic Token，选择 Repo

复制新生成的 Token

到仓库的 Settings，点击 Secrets an variables 下的 `Actions`，点击 `New repository secret`

Name 为 `ACCESS_TOKEN`，Secret 为刚刚复制的 Token

![](https://cdn.wallleap.cn/img/pic/illustration/202308101731398.png)

## 新增 Actions 文件

在仓库中新建目录 `.github/workflows`

在下面新建文件 `add-issue.yml`，内容为：

```yml
name: Add Issue
on:
  push:
    branches:
      - main
    paths:
      - '*.md'
      - '**/*.md'

jobs:
  readFiles:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '12'

      - name: Install front-matter
        run: npm install front-matter

      - name: Install axios
        run: npm install axios

      - name: Get Added Files
        id: files
        uses: jitterbit/get-changed-files@v1
        with:
          format: 'csv'

      - name: List all added files
        run: |
          mapfile -d ',' -t added_files < <(printf '%s,' '${{ steps.files.outputs.added }}')
          for added_file in "${added_files[@]}"; do
            echo "Do something with this ${added_file}."
            body=$(cat "${added_file}")
            body_json=$(printf '%s' "${body}" | sed 's/"/\\"/g' | jq -Rs '.' | tr -d '\n')
            title=$(node -e "const fs = require('fs'); const fm = require('front-matter'); const content = fs.readFileSync('${added_file}', 'utf8'); const parsed = fm(content); console.log(parsed.attributes.title);")
            labels=$(node -e "const fs = require('fs'); const fm = require('front-matter'); const content = fs.readFileSync('${added_file}', 'utf8'); const parsed = fm(content); console.log(parsed.attributes.labels);")
            echo "title: ${title}, body: ${body_json}, labels: ${labels}."
            echo "Sending POST request..."
            response=$(node -e "const axios = require('axios'); axios.post('https://api.github.com/repos/你的用户名/现在的仓库名/issues', {
              title: '${title}',
              assignees: [
                '你的用户名'
              ],
              body: ${body_json},
              labels: ${labels}
            }, {
              headers: {
                'Accept': 'application/vnd.github.v3+json',
                'Authorization': 'Bearer ${{ secrets.ACCESS_TOKEN }}',
                'X-GitHub-Api-Version': '2022-11-28'
              }
            }).then(res => {
              console.log(res.data);
            })
            .catch(err => {
              console.log(err);
            })")
            echo "Response: ${response}"
          done
```

![](https://cdn.wallleap.cn/img/pic/illustration/202308101734764.png)

其中中文【用户名】、【仓库名】需要替换成你自己的

例如我的是

```yml
response=$(node -e "const axios = require('axios'); axios.post('https://api.github.com/repos/oicoder/blog/issues', {
  title: '${title}',
  assignees: [
    'oicoder'
  ],
  body: ${body_json},
  labels: ${labels}
},
```

添加好了之后只要提交了 `*.md` 或 `**/*.md`（例如 `a.md`、`测试/b.md`）就会触发 Actions

## 提交自己的 md 文件

文件必须包含这两个 Front-matter

```yml
---
title: issue 标题
labels:
    - 标签1
    - 标签2
---
```

其中 title 任意，labels 必须先在 issue 中创建好，然后再填到这里

其他的 Front-matter 自行添加，如

```yml
---
title: CSS 学习
date: 2023-08-07 09:39
updated: 2023-08-07 09:40
cover: //cdn.wallleap.cn/img/post/1.jpg
labels:
    - CSS
    - auto
---

这里是博客的正文，Markdown 格式……
```

提交的命令

```sh
git add file.md
git commit -m "add file"
git push
```


随着 wordpress 和静态网站的流行，markdown 被用的越来越多。我们已经介绍过很多 Markdown 编辑器，但是有时候你也需要用程序来处理 Markdown 文本。

markdown 是一个面向写作的语法引擎，markdown 的最终目的都是解析成 html 用于网页浏览，所以它兼容 html 语法，即你可以在 markdown 文档中使用原生的 html 标签。

## markdown 解析器

开发静态网站生成器的时候都会采用一种叫 `front matter` 的格式进行网站内容写在类似下面的格式

```yaml
---
title: 玩转markdown，你需要用到这几个工具
date: 2016-08-14 16:44:54
image: /img/pencils-762555_640.jpg
---

## 前言

随着wordpress和静态网站的流行，markdown被用的越来越多。...
```

当进行网站生成的时候需要进行 `markdown` 解析，然后渲染成 `html` 页面，那用什么工具进行解析呢？

### **[marked](https://github.com/chjj/marked)**

`marked` 是最早用 `node.js `开发的 `markdown` 解析器，同时提供 `CLI` 命令调用和 `node.js API` 调用。

`CLI` 调用代码示例

```shell
$ marked -o hello.htmlhello world^D$ cat hello.html<p>hello world</p>
```

`API` 调用示例

```javascript
var marked = require('marked');
console.log(marked('I am using __markdown__.'));
// Outputs: <p>I am using <strong>markdown</strong>.</p>
```

这些都是一些通用的功能，但是 `marked` 还支持代码高亮，通过使用 [highlight.js](https://github.com/isagalaev/highlight.js)。

使用 `highlight.js` 进行代码高亮相信大家都用到过，可能大家不知道是 `highlight.js` 还支持 `API` 方式调用，下面的代码会配置 `marked` 使用 `highlight.js` 进行代码高亮：

```javascript
marked.setOptions({
	highlight: function (code, lang) {    
		var res;    
		if (lang) {      
			res = hljs.highlight(lang, code, true).value;    
		} else {      
			res = hljs.highlightAuto(code).value;    
		}    
		return res;  
	}
});
```

生成的代码已经包含代码高亮标签，最后只需要引入 `highlight.js` 的主题就能显示了，`highlight.js` 所有的颜色主题都在[这里](https://github.com/isagalaev/highlight.js/tree/master/src/styles)

### **[markdown-js](https://github.com/evilstreak/markdown-js)**

`markdown-js` 也是一款使用 `node.js` 开发的 `markdown` 解析器，基本用法和 `marked` 差不多，但是文档里面好像没有提到像 `marked` 一样进行代码高亮生成的接口，有兴趣的同学自己找找吧。

## markdown 生成器

### **[to-markdown](https://github.com/domchristie/to-markdown)**

什么是 `markdown` 生成器，就是根据 `html` 标签生成 `markdown` 文件。

`github` 上面 `markdown` 生成器 `star` 数最高的是 `to-markdown`。

简单的代码示例

```javascript
var toMarkdown = require('to-markdown');
toMarkdown('<h1>Hello world!</h1>');
```

`to-markdown` 最近进行了更新，增加了对 `gfm` 的兼容，`gfm` 就是 [git flavored markdown](https://guides.github.com/features/mastering-markdown/) 的意思， 是 `github` 对 `markdown` 语法进行的扩展。

使用 `gfm` 的示例

```javascript
toMarkdown('<del>Hello world!</del>', { gfm: true });
```

那这个 `to-markdown` 有什么用呢？

举个简单的例子，假如我想开发一个简单的 `RSS` 阅读器，但是我又不想跳转到目标网站去阅读，因为不同的网站风格不一，导致不一致的阅读体验。

怎么办呢？那就把网站内容抓取下来，然后用 `to-markdown` 生成 `markdown` 文件，然后使用自己的模板样式进行统一渲染。

当然去除广告只是一个 `side effect`。

### **[heckyesmarkdown](http://heckyesmarkdown.com/)**

除了 `to-markdown` 之外还有一个比较好用的 `API`： `heckyesmarkdown`，这个项目使用了 [php-readability](https://github.com/feelinglucky/php-readability)，提高文章的可读性。

可惜 `heckyesmarkdown` 没有开源出来，这个项目有点古老，估计那个时候 `github` 还没流行起来。

heckyesmarkdow 对中文的支持不是非常友好，如果想抓取中文站还是使用 `to-markdown` 比较靠谱一点。

## front matter

`markdown` 写文章确实很方便，简单容易上手，但是 `markdown` 不能保存元数据，例如作者，日期，类型这样的结构化的数据，如果都生成 html 标签的话提取的时候又稍微麻烦了点， 还得借助 [cheerio](https://github.com/cheeriojs/cheerio) 才能完成。

所以，为了能方便的保存文章的元数据，几乎所有的静态网站生成器都使用 `front matter` 格式来保存文章。

`front matter` 文件通常分为头部和正文部分，头部一般使用 [yaml](https://www.yaml.org/)、[toml](https://github.com/toml-lang/toml) 和 `json` 三种格式，`front matter` 解析工具需要识别这三种格式的文件头。正文部分就是普通的 `markdown` 内容。

### **[front-matter](https://github.com/jxson/front-matter)**

`front-matter` 也是用 `node.js `开发的，相比 `markdown` 解析器来说，`fornt-matter` 解析器要简单很多。

示例文件 `example.md`

```yaml
---
title: Just hack'n
description: Nothing to see here
---

This is some text about some stuff that happened sometime ago
```

解析代码

```javascript
var fs = require('fs')  
, fm = require('front-matter')
fs.readFile('./example.md', 'utf8', function(err, data){  
    if (err) throw err  
    var content = fm(data)  
    console.log(content)
})
```

解析结果

```json
{
    attributes: {
        title: 'Just hack\'n',
        description: 'Nothing to see here'
    },
    body: '\nThis is some text about some stuff that happened sometime ago',    frontmatter: 'title: Just hack\'n\ndescription: Nothing to see here'
}
```

`front matter` 虽然格式看起来不太统一，却是对 `markdown` 强有力的补充。
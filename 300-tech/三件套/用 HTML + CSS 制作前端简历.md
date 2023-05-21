
> 我观看了大量[饥人谷前端模拟面试锦集](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fxiedaimala.com%2Fcourses%2Fec472c6f-4a87-4983-9609-2f9ae5674c43%2Frandom%2F89dbe7185b%23%2Fcommon)， 观摩了优秀简历的细节 取他们亮点制作成了自己的简历

[吴思里的在线简历](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fwu-sili.gitee.io%2Fresume%2F)

[闵聪学长的在线简历](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttps%3A%2F%2Fresume.congm.in%2F)

[方应杭老师演示时制作的Demo](https://link.juejin.cn?target=https%3A%2F%2Flink.zhihu.com%2F%3Ftarget%3Dhttp%3A%2F%2Ffangyinghang.com%2Fcv-2020%2Fdist%2Findex.html%3Fv%3D1%23)

### **细节的点**

### **代码块效果**

修改strong默认样式，拥有`代码块`效果

```css
strong {
  font-family: 'Helvetica Neue', Helvetica, 'PingFang SC', 'Microsoft YaHei', '微软雅黑', Arial, sans-serif;
  font-size: 13px;
  line-height: 15px;
  font-weight: 500;
  color: #494949;
  margin: 0 3px;
  padding: 3px 8px;
  background-color: #F6F6F6;
  border-radius: 5px;
  border: 1px solid rgb(235, 235, 235);
}
```

使用时将代码块用`<strong>`标签包住即可

```html
熟悉<strong>HTML</strong>、<strong>JS(TS)</strong>、<strong>AJAX</strong>、<strong>ES6</strong>
```

### **修改滚动条**

```css
::-webkit-scrollbar {
  width: 5px;
  background-color: #f1f1f1;
}

::-webkit-scrollbar-thumb {
  background-color: rgba(0, 0, 0, 0.2);
  border-radius: 50px;
}
```

### **打印**

**A4尺寸为21cm\*29.7cm**

所以简历每页的大小比例相同即可

```css
.page{
    width: 1024px;
    min-height: 1440px;
}
```

**简历不只一页，不该断的地方在分页处了怎么办？**

标签中添加media属性 值为“print”，说明打印时才生效的样式 CSS `page-break-before`避免分页时内容的断开 

 ```html
 <style media="print">    .page2{        page-break-before:always;    } ... 
 ```

### **PDF简历**

开始还不知道右键打印可以网页另存为PDF

一开始还傻傻的先做word简历，

再用HTML把简历给写出来

说实话前端出身操作HHTML+CSS可比操作office擅长多了

**附上pdf简历下载链接**

在a标签上添加`download`属性就可以点击下载

```html
<a class="pdf" href="resume.pdf" download>
```

### **简历可编辑**

**怎么让别人拿你的简历改了就可以用?**

修改`designMode`属性

```js
document.designMode='on'
```

### **响应式**

适配不同宽度的效果

```css
@media screen and (max-width:1024px) {}
@media screen and (max-width: 720px) {}
```

作者：吴思里
链接：https://juejin.cn/post/6955061425161109540
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





[自学前端简历怎么写啊？ - 知乎 (zhihu.com)](https://www.zhihu.com/question/319340351/answer/645507315)


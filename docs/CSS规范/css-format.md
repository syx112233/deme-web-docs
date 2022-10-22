# 代码格式

### 1.选择器、属性和值都使用小写

在 xhtml 标准中规定了所有标签、属性和值都小写，CSS 也是如此。

### 2.单行写完一个选择器定义

便于选择器的寻找和阅读，也便于插入新选择器和编辑，便于模块等的识别。去除多余空格，使代码紧凑减少换行。

如果有嵌套定义，可以采取内部单行的形式。

```css
/* 单行定义一个选择器 */
.m-list li,
.m-list h3 {
  width: 100px;
  padding: 10px;
  border: 1px solid #ddd;
}
/* 这是一个有嵌套定义的选择器 */

@media all and (max-width: 600px) {
  .m-class1 .itm {
    height: 17px;
    line-height: 17px;
    font-size: 12px;
  }
  .m-class2 .itm {
    width: 100px;
    overflow: hidden;
  }
}
@-webkit-keyframes showitm {
  0% {
    height: 0;
    opacity: 0;
  }
  100% {
    height: 100px;
    opacity: 1;
  }
}
```

### 3.最后一个值也以分号结尾

通常在大括号结束前的值可以省略分号，但是这样做会对修改、添加和维护工作带来不必要的失误和麻烦。

### 4.省略值为 0 时的单位

为节省不必要的字节同时也使阅读方便，我们将 0px、0em、0%等值缩写为 0。

```css
.m-box {
  margin: 0 10px;
  background-position: 50% 0;
}
```

### 5.使用单引号

省略 url 引用中的引号，其他需要引号的地方使用单引号。

```css
.m-box {
  background: url(bg.png);
}
.m-box:after {
  content: '.';
}
```

### 6.使用 16 进制表示颜色值

除非你需要透明度而使用 rgba，否则都使用#f0f0f0 这样的表示方法，并尽量缩写。

```css
.m-box {
  color: #f00;
  background: rgba(0, 0, 0, 0.5);
}
```

### 7.根据属性的重要性按顺序书写

只遵循横向顺序即可，先显示定位布局类属性，后盒模型等自身属性，最后是文本类及修饰类属性。<br>

|  显示属性  | 自身属性  | 文本属性和其他修饰 |
| :--------: | :-------: | :----------------: |
|  display   |   width   |        font        |
| visibility |  height   |     text-align     |
|  position  |  margin   |  text-decoration   |
|   float    |  padding  |   vertical-align   |
|   clear    |  border   |    white-space     |
| list-style | overflow  |       color        |
|    top     | min-width |     background     |

<br>

```css
.m-box {
  position: relative;
  width: 600px;
  margin: 0 auto 10px;
  text-align: center;
  color: #000;
}
```

如果属性间存在关联性，则不要隔开写。

```css
/* 这里的height和line-height有关联性 */
.m-box {
  position: relative;
  height: 20px;
  line-height: 20px;
  padding: 5px;
  color: #000;
}
```

### 8.私有在前，标准在后

先写带有浏览器私有标志的，后写 W3C 标准的。

```css
.m-box {
  -webkit-box-shadow: 0 0 0 #000;
  -moz-box-shadow: 0 0 0 #000;
  box-shadow: 0 0 0 #000;
}
```

### 9.注释格式：/_ 注释文字 _/

对选择器的注释统一写在被注释对象的上一行，对属性及值的注释写于分号后。

注释内容两端需空格，已确保即使在编码错误的情况下也可以正确解析样式。

在必要的情况下，可以使用块状注释，块状注释保持统一的缩进对齐。

原则上每个系列的样式都需要有一个注释，言简意赅的表明名称、用途、注意事项等。

```css
/* 块状注释文字
 * 块状注释文字
 * 块状注释文字
 */
.m-list {
  width: 500px;
}
.m-list li {
  height: 20px;
  line-height: 20px; /* 这里是对line-height的一个注释 */
  overflow: hidden;
}
.m-list li a {
  color: #333;
}
/* 单行注释文字 */
.m-list li em {
  color: #666;
}
```

### 10.选择器顺序

请综合考虑以下顺序依据：

- 从大到小（以选择器的范围为准）<br>
- 从低到高（以等级上的高低为准）<br>
- 从先到后（以结构上的先后为准）<br>
- 从父到子（以结构上的嵌套为准）<br>

以下仅为简单示范：

```css
/* 从大到小 */
.m-list p {
  margin: 0;
  padding: 0;
}
.m-list p.part {
  margin: 1px;
  padding: 1px;
}
/* 从低到高 */
.m-logo a {
  color: #f00;
}
.m-logo a:hover {
  color: #fff;
}
/* 从先到后 */
.g-hd {
  height: 60px;
}
.g-bd {
  height: 60px;
}
.g-ft {
  height: 60px;
}
/* 从父到子 */
.m-list {
  width: 300px;
}
.m-list .itm {
  float: left;
}
```

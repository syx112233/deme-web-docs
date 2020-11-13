# HTML规范

## 1.HTML基础设施
* 文件应以```<!DOCTYPE ......>```首行顶格开始，推荐使用```<!DOCTYPE html>```。
* 必须申明文档的编码charset，且与文件本身编码保持一致，推荐使用UTF-8编码<meta charset="utf-8"/>。
* 网页端根据页面内容和需求填写适当的keywords和description。
* 页面title是极为重要的不可缺少的一项。<br>
例如：<br>
```
<!DOCTYPE html>
<html>
  <head>
  <meta charset="utf-8"/>
  <title>目标狗，你值得拥有的管理工具</title>
  <meta name="keywords" content=""/>
  <meta name="description" content=""/>
  <meta name="viewport" content="width=device-width"/>
  <link rel="stylesheet" href="css/style.css"/>
  <link rel="shortcut icon" href="img/favicon.ico"/>
  <link rel="apple-touch-icon" href="img/touchicon.png"/>
  </head>
  <body>
  </body>
</html>
```

### 2.结构顺序和视觉顺序基本保持一致
* 按照从上至下、从左到右的视觉顺序书写HTML结构。
* 有时候为了便于搜索引擎抓取，我们也会将重要内容在HTML结构顺序上提前。
* 用div代替table布局，可以使HTML更具灵活性，也方便利用CSS控制。
* table不建议用于布局，但表现具有明显表格形式的数据，table还是首选。

### 3.结构、表现、行为三者分离，避免内联
* 使用link将css文件引入，并置于head中。
* 使用script将js文件引入，并置于body底部。
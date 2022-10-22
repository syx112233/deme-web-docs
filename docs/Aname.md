# 命名规范

### 1.文件夹命名规范

通常情况下以功能命名，单个单词， 例如：upload、utils。
如多个单词，以-连接，例如：file-upload

### 2.文件命名规范

通常情况下以功能和统一入口命名，例如：upload.js、upload.ts。
如多个单词，以-连接， 例如：upload-file.js<br>
<font color="deepskyblue">React</font> 和 <font color="deepskyblue">Vue</font>组件名称和定义该组件的文件名称建议要保持一致<br>
推荐：

```
import Footer from './Footer';
```

  <br>
  框架具体命名规范会在具体章节提到。

### 3.目录结构规范

> 项目名称
>
> > config //配置文件<br>
> > src //项目代码文件<br> > > &nbsp;&nbsp;&nbsp;&nbsp;index.js //入口文件<br> > > &nbsp;&nbsp;&nbsp;&nbsp;...<br>
> > utils //工具类<br>
> > assets //静态资源<br>
> > release //生产环境打包输出<br>
> > dist //开发环境临时打包输出<br>
> > build //打包配置<br>
> > package.json //打包依赖配置文件<br>
> > tsconfig.ts // ts 配置文件<br>
> > ...

---
title: angular pdf 浏览插件ng2-pdfjs-viewer
date: 2020-12-28 19:00:00
categories: angular
tags:
---

pdf.js是mozilla公司的一个功能强大的pdf浏览库。下面介绍其在angular项目中的使用……

## ng2-pdfjs-viewer插件下载

[网址](https://github.com/intbot/ng2-pdfjs-viewer)

[使用案例](https://github.com/intbot/ng2-pdfjs-viewer/tree/master/SampleApp)

## 使用

具体，按照官网介绍以及查看demo就可以知道怎么用了，这里不过多介绍。

## 无法正常显示电子章的问题处理

1.下载源码，在`dist`目录下，新建文件夹`package`,把其它文件都放入`package`目录中。

2.编辑文件`E:\angular\ng2-pdfjs-viewer-master\dist\package\pdfjs\build\pdf.worker.js`,把下面一段注释掉：

```js
if (data.fieldType === 'Sig') {
  data.fieldValue = null;

  _this3.setFlags(_util.AnnotationFlag.HIDDEN);
}
```

3.把修改后的js压缩。百度js压缩工具，然后把压缩后的js内容，整个替换`pdf.worker.min.js`的内容。

4.在linux环境下，把整个`package`包压缩成`ng2-pdfjs-viewer-5.0.x.tgz`,压缩命令如下：

```shell script
tar -cvzf ng2-pdfjs-viewer-5.0.7.tgz package/
```

5.把压缩文件`ng2-pdfjs-viewer-5.0.7.tgz`拉下来放到angular项目的根目录下，如下：

{%asset_img a-1.png%}

最后在`package.json`文件中(看上面切图)引入。

这样，就可以和上面一样的使用自己修改代码后的版本了。



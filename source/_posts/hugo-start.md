---
title: 静态网站生成工具-Hugo
date: 2020-06-29 13:01:42
categories: hugo
tags:
---

Hugo是一个静态网站生成程序，类似Hexo。但是Hugo是由Go语言编写的，完全开源、高性能、安全、可高定制化、具有丰富的模板……

[官网](https://gohugo.io/)    [源码](https://github.com/gohugoio)

## Hugo安装

### Windows

#### 二进制文件安装（推荐）

1.官方源码库下载编译好的二进制可执行文件

https://github.com/gohugoio/hugo/releases

{% asset_img a-1.png %}

2.解压并放到目录`C:\Hugo\bin`下：

{% asset_img a-2.png %}

3.把可执行文件路径添加到环境变量，以便在任何路径下都可以访问

{% asset_img a-3.png %}

4.查看安装是否成功

随便打开命令行执行下面命令：

```shell script
C:\Users\xinxiamu>hugo version
Hugo Static Site Generator v0.73.0-428907CC windows/amd64 BuildDate: 2020-06-23T16:32:10Z
```
看到打印出`hugo`的安装版本号，那就说明安装成功了。

#### 源码编译安装

不推荐，需要翻墙，否则编译的时候有些依赖的包无法下载，无法编译成功。

1.安装golang语言环境。

2.把`hugo`作为一个go项目编译，即能得到对应的二进制可执行文件。

## 快速使用(win10环境)

### 第一步：创建一个新的站点

打开某个目录，执行：

```shell script
F:\>hugo new site quickstart
Congratulations! Your new Hugo site is created in F:\quickstart.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/ or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>\<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
```

将会在该目录下创建命为`quickstart`的目录，就是静态网站项目的目录。

### 第二步：添加主题

主题商店：[themes.gohugo.io](https://themes.gohugo.io/)

下面使用炫酷的主题[Ananke theme](https://themes.gohugo.io/gohugo-theme-ananke/)

```shell script
cd quickstart
git init
git submodule add https://github.com/budparr/gohugo-theme-ananke.git themes/ananke
```

{% asset_img b-1.png %}

添加主题到配置文件：

```shell script
echo 'theme = "ananke"' >> config.toml
```

{% asset_img b-2.png %}

### 第三步：添加内容

```shell script
hugo new posts/my-first-post.md
```
{% asset_img b-3.png %}

会在目录`content`创建`posts`目录，并在`posts`创建文件`my-first-post.md`。

下面你就可以编辑该内容文件啦……

### 第四步：启动服务

现在，[drafts](https://gohugo.io/getting-started/usage/#draft-future-and-expired-content)草案模式启动服务:

```shell script
C:\Users\xinxiamu\quickstart>hugo server -D
Building sites ...
                   | EN
-------------------+-----
  Pages            | 10
  Paginator pages  |  0
  Non-page files   |  0
  Static files     |  6
  Processed images |  0
  Aliases          |  1
  Sitemaps         |  1
  Cleaned          |  0

Built in 50 ms
Watching for changes in C:\Users\xinxiamu\quickstart\{archetypes,content,data,layouts,static,themes}
Watching for config changes in C:\Users\xinxiamu\quickstart\config.toml
Environment: "development"
Serving pages from memory
Running in Fast Render Mode. For full rebuilds on change: hugo server --disableFastRender
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
Press Ctrl+C to stop
```

在浏览器打开：

http://localhost:1313/

### 第五步：定制主题

上面生成的静态网站看起来已经很不错了，但是在实际发布之前，看样子你还需要稍微调整下。

网站配置：

打开文件`config.toml`并编辑：

```text
baseURL = "https://example.org/"
languageCode = "en-us"
title = "My New Hugo Site"
theme = "ananke"
```

>编辑文件，浏览器马上会刷新并能看到效果，无需刷新或者清除浏览器缓存。因为Hugo集成了reloadLive。publishdir 


关于该主题的一些其他的特别的配置，请参考[ theme site](https://github.com/theNewDynamic/gohugo-theme-ananke)

进一步的主题定制，请参考[主题定制](https://gohugo.io/hugo-modules/theme-components/)

### 第六步：生成静态页面

很简单，只需要执行命令：

```shell script
hugo -D
```

`-D`: 静态页面输出的目录。

默认情况下，输出在目录`./public/`。但是你可以通过`-d/--destination`指定输出目录，或者在配置文件中设置`publishdir`参数。

>Drafts(草稿)，该模式下的内容是不会发布的，因此，但你的内容文件编辑好了，需要发布的时候，必须要把draft: true 改为 draft: false。更多内容参考[这里](https://gohugo.io/getting-started/usage/#draft-future-and-expired-content)




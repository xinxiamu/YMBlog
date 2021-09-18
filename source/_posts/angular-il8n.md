---
title: Angular 国际化
date: 2021-09-18 10:17:37
categories: angular
tags:
---

本文介绍angular项目中，如何做国际化显示。

`angular version： 12.1.0`

## 引入国际化相关依赖

在依赖管理文件`package.json`中引入国际化相关依赖：

{% asset_img a_1.png %}

## 配置模块信息

在`app.module.ts`文件中配置如下信息：

{% asset_img b_1.png %}

另外注意导入配置：

{% asset_img b_2.png %}

## 初始化国际化信息

在`angular`项目入口组件文件`app.component.ts`文件中做初始化：

```shell
constructor(private translate: TranslateService) {
    
    // 国际化
    translate.addLangs(['en', 'zh-CN', 'zh-CNT']);
    translate.setDefaultLang('zh-CN');
    // 默认设置跟随浏览器语言
    const browserLang = translate.getBrowserLang();
    // 按文件名匹配使用的国际化配置文件
    translate.use(browserLang.match(/en|zh-CN|zh-CNT/) ? browserLang : 'zh-CN');
  }
```

## 添加国际化配置文件

最后，别忘记了要添加国际化配置文件哦：

{% asset_img c_1.png %}

配置文件内容比如如下：

```json
{
  "title": "IT eBooks",
  "translate": "Language",
  "home": "All IT eBooks",
  "categories": "Categories",
  "language": {
    "en": "English",
    "zh": "Chinese",
    "zht": "Chinese(Traditional)"
  },
  "search": "search",
  "book-details":{
    "desc": "Book Description",
    "comment": "Comment",
    "comment-form": {
      "comment-control": {
        "nzErrorTip": "Please write something here!",
        "placeholder": "write any thing"
      },
      "name-control": {
        "nzErrorTip": "Please write write your name",
        "placeholder": "write your name"
      },
      "email-control": {
        "nzErrorTip": "The input is not valid E-mail!",
        "required": "Please input your E-mail!",
        "placeholder": "email"
      }
    },
    "related": "Related eBooks"
  },
  "download": "Download",
  "submit": "submit",
  "footer-text": "Reproduction of site books is authorized only for informative purposes and strictly for personal, private use.     © 2019 IT eBooks"
}

```

## 使用国际化

#### 在代码中使用

```ts
this.translate.get("book-details.desc").subscribe(value => {
      console.log(value);
    });

this.translate.get("title").subscribe(value => {
  console.log(value);
});
```

#### 在`html`页面中使用

首先，要在所在组件中引入国际化服务：

```text
constructor(public translate: TranslateService) {}
```

页面中使用：

```text
<div class="all-it-ebook">
    <span>{{'home' | translate}}</span>
</div>
```

关键代码：
```text
{{'home' | translate}}
```

另外：
```text
<span *ngIf="translate.currentLang === 'en'">{{'language.en' | translate}}</span>
<span *ngIf="translate.currentLang === 'zh-CN'">{{'language.zh' | translate}}</span>
<span *ngIf="translate.currentLang === 'zh-CNT'">{{'language.zht' | translate}}</span>
```

------------------------------------------------------------------------------------------------

完！！！！！！！！！！！！！！！



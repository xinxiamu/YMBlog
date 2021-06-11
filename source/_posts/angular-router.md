---
title: angular路由与导航
date: 2019-11-10 11:03:49
categories: angular
tags:
---

在用户使用应用程序时，Angular 的路由器能让用户从一个视图导航到另一个视图。   
浏览器具有熟悉的导航模式：

- 在地址栏输入 URL，浏览器就会导航到相应的页面。
- 在页面中点击链接，浏览器就会导航到一个新页面。
- 点击浏览器的前进和后退按钮，浏览器就会在你的浏览历史中向前或向后导航。


## 基础知识

#### <base href> 元素

大多数带路由的应用都要在index.html的 <head> 标签下先添加一个 <base> 元素，来告诉路由器该如何合成导航用的 URL。

设置src/index.html (base-href)

     <!DOCTYPE html>
     <html lang="en">
       <head>
         <!-- Set the base href -->
         <base href="/">
         <title>Angular Router</title>
         <meta charset="UTF-8">
         <meta name="viewport" content="width=device-width, initial-scale=1">
       </head>
     
       <body>
         <app-root></app-root>
       </body>
     
     </html>
     
#### 导入路由模块

在src/app/app-routing.module.ts 导入路由模块

    import { RouterModule, Routes } from '@angular/router';
    
#### 配置路由

在src/app/app-routing.module.ts (excerpt)中配置路由：

    const appRoutes: Routes = [
      { path: 'crisis-center', component: CrisisListComponent },
      { path: 'hero/:id',      component: HeroDetailComponent },
      {
        path: 'heroes',
        component: HeroListComponent,
        data: { title: 'Heroes List' }
      },
      { path: '',
        redirectTo: '/heroes',
        pathMatch: 'full'
      },
      { path: '**', component: PageNotFoundComponent }
    ];
    
    @NgModule({
      imports: [
        RouterModule.forRoot(
          appRoutes,
          { enableTracing: true } // 测试环境才打开
        )
        // other imports here
      ],
      ...
    })
    export class AppModule { }
    
以上定义一个路由器数组，里面五个路由信息。并用 RouterModule.forRoot() 方法来配置路由器， 并把它的返回值添加到 AppModule 的 imports 数组中。 

这里的路由数组 appRoutes 描述如何进行导航。 把它传给 RouterModule.forRoot() 方法并传给本模块的 imports 数组就可以配置路由器。    

每个路由都会把一个URL的path映射到一个具体的组件。注意，path不能以`斜杆`开头。 路由器会为解析和构建最终的 URL，这样当你在应用的多个视图之间导航时，可以任意使用相对路径和绝对路径。

第二个路由中的:id是一个路由参数，后面章节会有更多介绍。

第三个路由中的 data 属性用来存放于每个具体路由有关的任意信息。该数据可以被任何一个激活路由访问，并能用来保存诸如 页标题、面包屑以及其它静态只读数据。如何获取这些存放的信息，后面介绍。 

第四个路由中的空路径（''）表示应用的默认路径，当 URL 为空时就会访问那里，因此它通常会作为起点。 这个默认路由会重定向到 URL /heroes。

最后一个路由中的 ** 路径是一个通配符。当所请求的 URL 不匹配前面定义的路由表中的任何路径时，路由器就会选择此路由。 这个特性可用于显示“404 - Not Found”页，或自动重定向到其它路由。

这些路由的定义顺序是刻意如此设计的。路由器使用先匹配者优先的策略来匹配路由，所以，具体路由应该放在通用路由的前面。在上面的配置中，带静态路径的路由被放在了前面，后面是空路径路由，因此它会作为默认路由。而通配符路由被放在最后面，这是因为它能匹配上每一个 URL，因此应该只有在前面找不到其它能匹配的路由时才匹配它。

如果你想要看到在导航的生命周期中发生过哪些事件，可以使用路由器默认配置中的 enableTracing 选项。它会把每个导航生命周期中的事件输出到浏览器的控制台。 这应该只用于调试。你只需要把 enableTracing: true 选项作为第二个参数传给 RouterModule.forRoot() 方法就可以了。

#### 路由出口

RouterOutlet是一个来自路由模块的一个指令，用法类似组件。它扮演一个占位符的角色，用于在模板中标出一个位置，路由将会把要显示在这个出口处的组件显示在这里。

    <router-outlet></router-outlet>
    <!-- Routed components go here -->
    
有了这份配置，当本应用在浏览器中的 URL 变为 /heroes 时，路由器就会匹配到 path 为 heroes 的 Route，并在宿主视图中的RouterOutlet之后显示 HeroListComponent 组件。

   
#### 路由器链接

在地址栏输入对应路由，能导航到相应页面。但多数时候，路由导航是用户操作的结果，查看下面例子：

    <h1>Angular Router</h1>
    <nav>
      <a routerLink="/crisis-center" routerLinkActive="active">Crisis Center</a>
      <a routerLink="/heroes" routerLinkActive="active">Heroes</a>
    </nav>
    <router-outlet></router-outlet>
    
a 标签上的 RouterLink 指令让路由器得以控制这个 a 元素。 这里的导航路径是固定的，因此可以把一个字符串赋给 routerLink（“一次性”绑定）。

#### 路由链接的激活状态

RouterLinkActive 指令会基于当前的 RouterState 为活动的 RouterLink 切换所绑定的 css 类。

#### 路由器状态

在导航时的每个生命周期成功完成时，路由器会构建出一个 ActivatedRoute 组成的树，它表示路由器的当前状态。 你可以在应用中的任何地方用 Router 服务及其 routerState 属性来访问当前的 RouterState 值。

RouterState 中的每个 ActivatedRoute 都提供了从任意激活路由开始向上或向下遍历路由树的一种方式，以获得关于父、子、兄弟路由的信息。

#### 激活的路由

该路由的路径和参数可以通过注入进来的一个名叫ActivatedRoute的路由服务来获取。

#### 路由事件

在每次导航中，Router 都会通过 Router.events 属性发布一些导航事件。这些事件的范围涵盖了从开始导航到结束导航之间的很多时间点。下表中列出了全部导航事件：

#### 总结

该应用有一个配置过的路由器。 外壳组件中有一个 RouterOutlet，它能显示路由器所生成的视图。 它还有一些 RouterLink，用户可以点击它们，来通过路由器进行导航。

下面是一些路由器中的关键词汇及其含义：

|  路由器部件   | 含义 |
| :------: |:-----:|
| Router（路由器） | 为激活的 URL 显示应用组件。管理从一个组件到另一个组件的导航 |
|RouterModule | 一个独立的 NgModule，用于提供所需的服务提供商，以及用来在应用视图之间进行导航的指令。 |
| Routes（路由数组） | 定义了一个路由数组，每一个都会把一个 URL 路径映射到一个组件。 |
| Route（路由） | 定义路由器该如何根据 URL 模式（pattern）来导航到组件。大多数路由都由路径和组件类构成。 |
| RouterOutlet（路由出口） | 该指令（<router-outlet>）用来标记出路由器该在哪里显示视图。 |
|RouterLink（路由链接）|这个指令把可点击的 HTML 元素绑定到某个路由。点击带有 routerLink 指令（绑定到字符串或链接参数数组）的元素时就会触发一次导航。|
|RouterLinkActive（活动路由链接）|当 HTML 元素上或元素内的routerLink变为激活或非激活状态时，该指令为这个 HTML 元素添加或移除 CSS 类。|
|ActivatedRoute（激活的路由）|为每个路由组件提供提供的一个服务，它包含特定于路由的信息，比如路由参数、静态数据、解析数据、全局查询参数和全局碎片（fragment）。|
|RouterState（路由器状态）|路由器的当前状态包含了一棵由程序中激活的路由构成的树。它包含一些用于遍历路由树的快捷方法。|
|链接参数数组|这个数组会被路由器解释成一个路由操作指南。你可以把一个RouterLink绑定到该数组，或者把它作为参数传给Router.navigate方法。|
|路由组件|一个带有RouterOutlet的 Angular 组件，它根据路由器的导航来显示相应的视图。|

## 范例应用

下面讲解如何开发一个带路由的多页面小应用。涉及到下面一些决策：

- 把应用的各个特性组织成模块。
- 导航到组件（Heroes 链接到“英雄列表”组件）。
- 包含一个路由参数（当路由到“英雄详情”时，把该英雄的 id 传进去）。
- 子路由（危机中心特性有一组自己的路由）。
- CanActivate 守卫（检查路由的访问权限）。
- CanActivateChild 守卫（检查子路由的访问权限）。
- CanDeactivate 守卫（询问是否丢弃未保存的更改）。
- Resolve 守卫（预先获取路由数据）。
- 惰性加载特性模块。
- CanLoad 守卫（在加载特性模块之前进行检查）。


学习参考：

- https://angular.cn/guide/router#base-href

-------------------------------------------------------------------

## 路由传参

#### 情景一：

路径：http://localhost:8080/#/product/1 

跳转传参：

方式一
```typescript
this.router.navigate(['/product/2']);
```

方式二： 
```html
<a [routerLink]="['/details', item.id]">
{{item.nameEn}}
</a>
```

获取参数值

方式一： 
```typescript
this.productId = this.routeInfo.snapshot.params['id'];
```

方式二：  
```typescript
this.routeInfo.params.subscribe((params: Params) => this.productId = params['id'])
```

#### 情景二

路径：http://localhost:8080/#/product?a=11&b=33

传参：
```typescript
this.router.navigate(['list'], {queryParams: {menuName: nameEn}});
```

获取参数：
```typescript
this._menuName = this.route.snapshot.queryParams.menuName;
```

#### Angualr routerLink 两种传参方法及参数的使用

1.路径：http://localhost:8080/#/product?id=1

```html
<a [routerLink]="['/product']" [queryParams]="{id:1}">详情</a>
```

```typescript
//获取参数值
this.productId = this.routeInfo.snapshot.queryParams['id'];
```

2.路径：http://localhost:8080/#/product/1

```html
<a [routerLink]="['/product',1]">产品</a>
```

```typescript
//获取参数值
this.productId = this.routeInfo.snapshot.params['id'];
 //另一种方式参数订阅
this.routeInfo.params.subscribe((params: Params) => this.productId = params['id']);
```

这种需要配置路由：

```typescript
const routes: Routes =[
    {path: 'product/:id',component: ProductComponent}
];
```

## 重新加载路由的问题

问题描述：   
>同一个路由，当前路由下，路由参数变化，不会重新加载页面，即不会重新执行onInit方法。

解决办法：   
在AppComponet添加如下代码：
```typescript
// 处理同一个路由，参数变化，不会重新加载界面的问题。
// tslint:disable-next-line:only-arrow-functions
this.router.routeReuseStrategy.shouldReuseRoute = function() {
    return false;
};
this.router.events.subscribe((evt) => {

    if (evt instanceof NavigationEnd) {
        this.router.navigated = false;
        window.scrollTo(0, 0);
    }
});
```



    

     
    

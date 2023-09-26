---
title: 使用angular开发chrome插件问题
date: 2023-05-23 13:04:13
categories: angular
tags: angular, chrome
---

##问题一：

angular使用了encapsulation: ViewEncapsulation.ShadowDom后，界面无法自动变更检测

```text
当你在 Angular 组件中设置 `encapsulation: ViewEncapsulation.ShadowDom` 后，组件的样式和模板将被封装在 Shadow DOM 中。这意味着组件的样式和 DOM 结构将与外部的样式和 DOM 相隔离，不会互相影响。

然而，封装在 Shadow DOM 中可能导致一些变更检测的问题，因为 Angular 的默认变更检测机制可能无法正确检测到 Shadow DOM 内部的变化。这可能导致界面无法自动更新。

解决这个问题的方法是使用 Angular 的 `NgZone` 服务来手动触发变更检测。`NgZone` 可以使 Angular 在特定的上下文中运行变更检测，包括 Shadow DOM 内部。

首先，将 `NgZone` 服务注入到组件的构造函数中：

```typescript
import { NgZone } from '@angular/core';

constructor(private ngZone: NgZone) { }
```

然后，在需要手动触发变更检测的地方，使用 `ngZone.run()` 方法来运行变更检测：

```typescript
this.ngZone.run(() => {
  // 在这里进行需要触发变更检测的操作
});
```

在你的代码中，可以在接收到消息并更新变量后，使用 `ngZone.run()` 方法来触发变更检测：

```typescript
chrome.runtime.onMessage.addListener((request, sender, sendResponse) => {
  console.log('Received message:', request);
  const type = request.type;
  if (type === 'action') {
    this.ngZone.run(() => {
      this.isShowMain = request.msg;
    });
  }
  sendResponse(`我是app-root，toggle右侧主框：${request.msg}`);
});
```

通过使用 `ngZone.run()` 方法，可以确保变更检测在正确的上下文中进行，从而使界面能够正确地更新。

请注意，使用 `ngZone.run()` 方法会触发整个组件及其子组件的变更检测，因此请避免在大型应用中滥用它，以免影响性能。

希望这能解决你的问题。如果还有其他疑问，请随时提问。
```

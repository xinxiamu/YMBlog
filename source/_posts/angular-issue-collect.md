---
title: Angular使用问题收集
date: 2019-12-01 18:05:10
categories: angular
tags:
---

## 问题一

问题描述：

引入第三方依赖包，如“buffer”，使用报错如下：

    core.js:6014 ERROR Error: Uncaught (in promise): ReferenceError: global is not defined
    ReferenceError: global is not defined
        at Object../node_modules/_buffer@4.9.2@buffer/index.js (index.js:43)
        at __webpack_require__ (bootstrap:84)
        at Module../src/app/pages/login/login.component.ts (pages-pages-module.js:108766)
        at __webpack_require__ (bootstrap:84)
        at Module../src/app/pages/pages-routing.module.ts (login.component.ts:14)
        at __webpack_require__ (bootstrap:84)
        at Module../src/app/pages/pages.module.ts (pages-routing.module.ts:36)
        at __webpack_require__ (bootstrap:84)
        at $_lazy_route_resource lazy namespace object:22
        at ZoneDelegate.invoke (zone-evergreen.js:359)
        at resolvePromise (zone-evergreen.js:797)
        at resolvePromise (zone-evergreen.js:754)
        at zone-evergreen.js:858
        at ZoneDelegate.invokeTask (zone-evergreen.js:391)
        at Object.onInvokeTask (core.js:39680)
        at ZoneDelegate.invokeTask (zone-evergreen.js:390)
        at Zone.runTask (zone-evergreen.js:168)
        at drainMicroTaskQueue (zone-evergreen.js:559)


解决：

在文件`src/polyfills.ts`中，添加一行`(window as any).global = window;`

如下：

    /**
     * This file includes polyfills needed by Angular and is loaded before the app.
     * You can add your own extra polyfills to this file.
     *
     * This file is divided into 2 sections:
     *   1. Browser polyfills. These are applied before loading ZoneJS and are sorted by browsers.
     *   2. Application imports. Files imported after ZoneJS that should be loaded before your main
     *      file.
     *
     * The current setup is for so-called "evergreen" browsers; the last versions of browsers that
     * automatically update themselves. This includes Safari >= 10, Chrome >= 55 (including Opera),
     * Edge >= 13 on the desktop, and iOS 10 and Chrome on mobile.
     *
     * Learn more in https://angular.io/guide/browser-support
     */
    
    /***************************************************************************************************
     * BROWSER POLYFILLS
     */
    
    /** IE10 and IE11 requires the following for NgClass support on SVG elements */
    // import 'classlist.js';  // Run `npm install --save classlist.js`.
    
    /**
     * Web Animations `@angular/platform-browser/animations`
     * Only required if AnimationBuilder is used within the application and using IE/Edge or Safari.
     * Standard animation support in Angular DOES NOT require any polyfills (as of Angular 6.0).
     */
    // import 'web-animations-js';  // Run `npm install --save web-animations-js`.
    
    /**
     * By default, zone.js will patch all possible macroTask and DomEvents
     * user can disable parts of macroTask/DomEvents patch by setting following flags
     * because those flags need to be set before `zone.js` being loaded, and webpack
     * will put import in the top of bundle, so user need to create a separate file
     * in this directory (for example: zone-flags.ts), and put the following flags
     * into that file, and then add the following code before importing zone.js.
     * import './zone-flags.ts';
     *
     * The flags allowed in zone-flags.ts are listed here.
     *
     * The following flags will work for all browsers.
     *
     * (window as any).__Zone_disable_requestAnimationFrame = true; // disable patch requestAnimationFrame
     * (window as any).__Zone_disable_on_property = true; // disable patch onProperty such as onclick
     * (window as any).__zone_symbol__UNPATCHED_EVENTS = ['scroll', 'mousemove']; // disable patch specified eventNames
     *
     *  in IE/Edge developer tools, the addEventListener will also be wrapped by zone.js
     *  with the following flag, it will bypass `zone.js` patch for IE/Edge
     *
     *  (window as any).__Zone_enable_cross_context_check = true;
     *
     */
    (window as any).global = window;
    
    /***************************************************************************************************
     * Zone JS is required by default for Angular itself.
     */
    import 'zone.js/dist/zone';  // Included with Angular CLI.
    
    
    /***************************************************************************************************
     * APPLICATION IMPORTS
     */

## 问题-绑定html内容出现警告

问题：

```html
<span [innerHTML]="{{detailsInfo.description}}"></span>

---------------------------------------

WARNING: sanitizing HTML stripped some content (see http://g.co/ng/security#xss).
```

解决：

```text
 constructor(public sanitizer: DomSanitizer) {
  }
```
```html
<span [innerHTML]="sanitizer.bypassSecurityTrustHtml(detailsInfo.description)"></span>
```



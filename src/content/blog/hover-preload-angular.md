---
title: 'Optimising Angular Performance with ngx-hover-preload'
description: 'Preload a lazy-loaded route on mouse over a corresponding router link'
pubDate: 'Jan 30 2021'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/cwgw8r3zkn5qzajm5stj.png'
---
Some time ago I wrote this post to understand [how to implement lazy loading and QuickLink as preloading strategy on Angular applications](https://dev.to/salimchemes/optimising-angular-performance-with-lazy-loading-and-quicklink-3j66). 

On this post, we will review a new preloading strategy called **ngx-hover-preload**. 

As you can imagine, the idea is to download our module once hover a routerlink. This new library was released on January by Minko Gechev. Let's take a look to see how it works.

_(If you have any questions or concerns about what lazy loading and preloading strategy, just look at [this](https://dev.to/salimchemes/optimising-angular-performance-with-lazy-loading-and-quicklink-3j66).)_


We will use a very simple app
* list of dogs (with lazy loading)
* list of cats (with lazy loading)

**1- Add ngx-hover-preload**
* `npm i ngx-hover-preload --save`

**2- Import HoverPreloadModule on app.module.ts**
```javascript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { HoverPreloadModule } from 'ngx-hover-preload';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, AppRoutingModule, HoverPreloadModule],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {} 
```

**3- Define HoverPreloadStrategy preloading strategy on app-routing.module.ts**
```javascript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HoverPreloadStrategy } from 'ngx-hover-preload';

const routes: Routes = [
  {
    path: 'dogs',
    loadChildren: () => import('./dogs/dogs.module').then((m) => m.DogsModule),
  },
  {
    path: 'cats',
    loadChildren: () => import('./cats/cats.module').then((m) => m.CatsModule),
    data: {
      preload: false,
    },
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: HoverPreloadStrategy }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}

```

So as you can see on app-routing.module.ts, cats module has **preload: false**. It means that no preloading strategy will apply on it. Since dogs module does no have it, by default, preloading strategy will be applied. You can use preload flag to opt-in/opt-out.

**4- Adds routerLink that points to the modules on templates (Dont Forget router-outlet)**

```javascript
<div>
  <a routerLink="">home</a>
</div>
<br />
<div>
  <a routerLink="dogs">dogs</a>
</div>
<br />
<div>
  <a routerLink="cats">cats</a>
</div>
<router-outlet></router-outlet>

```


Let's it see on action
**Hover on dogs routerlink (preloading strategy enabled, module is downloaded)**
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/t0jro6qp58e9yzsew9t9.gif)
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/bt0oqy50ebhgyre89wyn.png)

**Hover on cats routerlink (preloading strategy disabled, module is not downloaded)**
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/2wsiqpj05c2n14h8yqrx.gif)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ncd7hke9eo7vi4ff3i6p.png)


**Conclusion**
With **ngx-hover-preload** library we can optimize the way we handle our Angular modules. This new preloading strategy can be very useful if you want to go an extra mile with performance improvements.

Thanks for reading

References
* [repo](https://github.com/salimchemes/lazy-loading-quicklink/tree/lazy-loading-ngx-hover-preload) with 2 feature modules 
      * 1 with preload: true. By default, no need to set it(dogs).
      * 1 with preload: false (cats)
* [ngx-hover-preload library by Minko Gechev](https://github.com/mgechev/ngx-hover-preload)

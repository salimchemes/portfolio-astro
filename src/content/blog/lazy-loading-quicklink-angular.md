---
title: 'Optimising Angular Performance with Lazy loading and Quicklink'
description: 'Optimising Angular Performance with Lazy loading and Quicklink'
pubDate: 'Aug 17 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/b0mqvhpg901uir7m6igb.png'
---

Performance is one of the most important things on any app.

Mostly when working on large projects, we will have multiple routes, complex business rules and a lot of code for sure.

By default, Angular compiles all our code in a single file called _main.js_. In small projects this is probably fine, but we usually work on enterprise applications so instead of having a single file, we can create one js for each feature module and download it when the route is invoked.

Let's see the difference between default loading and lazy loading feature modules

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/d3akqwtrloyafuaaaqw1.png)

Angular provides lazy-loading to help us to achieve this goal. In this post we will
* **Implement lazy-loading feature modules**
* **Implement ngx-quicklink**: router preloading strategy which automatically downloads the lazy-loaded modules associated with all the visible links on the screen **(any routerLink found will be downloaded)**. This is not part of Angular, it's a [library](https://github.com/mgechev/ngx-quicklink) written by Minko Gechev.

Let's get started with our example, a simple app with 2 feature modules
* list of dogs
* list of cats

```javascript
 ng new lazy-loading-quicklink
```

Now, we are going to create the first feature module with lazy loading

```javascript
ng generate module dogs --route dogs --module app.module

```
as result we have
* created new dogs.module.ts
```javascript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { DogsRoutingModule } from './dogs-routing.module';
import { DogsComponent } from './dogs.component';

@NgModule({
  declarations: [DogsComponent],
  imports: [CommonModule, DogsRoutingModule],
})
export class DogsModule {}

```
* created new dogs-routing.module.ts
```javascript
import { NgModule } from '@angular/core';
import { Routes, RouterModule } from '@angular/router';

import { DogsComponent } from './dogs.component';

const routes: Routes = [{ path: '', component: DogsComponent }];

@NgModule({
  imports: [RouterModule.forChild(routes)],
  exports: [RouterModule],
})
export class DogsRoutingModule {}

```
* updated app-routing.module.ts including new dogs routing (note that lazy loading is in place)
```javascript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: 'dogs',
    loadChildren: () => import('./dogs/dogs.module').then((m) => m.DogsModule),
  }
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}

```
* created new dogs.component

Another important step is to include **router-outlet** in the **app.component.html** to be able to perform routing actions
```javascript

<div>
  <a routerLink="">home</a>
</div>
<div>
  <a routerLink="dogs">dogs</a>
</div>
<router-outlet></router-outlet>

```
So, we have our feature module, let's now implement ngx-quicklink

* `npm i ngx-quicklink --save`

Once it's done, we need to update app.module.ts to import **QuicklinkModule**
```javascript

import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { QuicklinkModule } from 'ngx-quicklink';

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, AppRoutingModule, QuicklinkModule],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```
Next step is to set the preloadingStrategy on **app-routing.module.ts**
```javascript

import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { QuicklinkStrategy } from 'ngx-quicklink';

const routes: Routes = [
  {
    path: 'dogs',
    loadChildren: () => import('./dogs/dogs.module').then((m) => m.DogsModule),
  },
];

@NgModule({
  imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: QuicklinkStrategy }),
  ],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

We have defined preloadingStrategy in our routing module, it  means when **routerLink="dogs"** is rendered, dogs module will be downloaded before hand.

Here we have some screenshots to show what is going on

routerLink is rendered
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/sc4xtp3kqymqbric8tcb.png)


dogs feature module is downloaded
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/hulo4yo4r6m9kypnu778.png)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/t47diah8pxblaf42w6hn.gif)

In case you don't want to prefetch a route, you can set **preload:false**

```javascript

const routes: Routes = [
  {
    path: 'dogs',
    loadChildren: () => import('./dogs/dogs.module').then((m) => m.DogsModule),
    data: {
      preload: false,
    },
  },
]; 
```
In that case, feature module will be downloaded once you access to the route and not before.

**Conclusion**
Lazy loading is a great technique to enhance performance splitting the code. If we want to go one step forward, ngx-quicklink helps us to improve even more adding prefetching feature modules. It's not hard to implement and performance result are much better.

Thanks for reading

References
* [repo](https://github.com/salimchemes/lazy-loading-quicklink/tree/master) with 2 feature modules 
      * 1 with preload: true. By default, no need to set it(dogs).
      * 1 with preload: false (cats)
* [lazy loading docs](https://angular.io/guide/lazy-loading-ngmodules)
* [ngx-quicklink library by Minko Gechev](https://github.com/mgechev/ngx-quicklink)
* [Tools for fast Angular applications](https://www.youtube.com/watch?v=l8mCutUMh78)

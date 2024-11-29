---
title: 'ResizeObserver in Angular'
description: 'ResizeObserver in Angular'
pubDate: 'Mar 17 2021'
heroImage: 'https://media2.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fj29zm8k2kc1ovbnwvbcd.png'
---

In this post we will review how to implement ResizeObserver in Angular applications

**What is ResizeObserver?**
Based on the [documentation](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver)

_The ResizeObserver interface reports changes to the dimensions of an Element's content or border box, or the bounding box of an SVGElement._

So, our goal is know when our component is resized.

**How can implement ResizeObserver in Angular?**
We will use [ng-web-apis/resize-observer](https://github.com/ng-web-apis/resize-observer)

The documentation is pretty clear, so let's start with an example to see how this library works.


First thing to do is to add `@ng-web-apis/common` as dependency
```
npm i @ng-web-apis/common
```

Second is to add 
```
npm i @ng-web-apis/resize-observer
```

and finally

```
npm install --save @types/resize-observer-browser
```

Now, let's start adding changes into our app.
Adding `ResizeObserverModule` into app.module.ts
```javascript
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { ResizeObserverModule } from '@ng-web-apis/resize-observer';
import { AppComponent } from './app.component';


@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    ResizeObserverModule // adding library module
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

Adding a section in app.component.html, to display resize changes.
Note we have **waResizeBox="content-box"**, which is the default value. Possible values are content-box (the default), border-box, and device-pixel-content-box. Check the [docs](https://developer.mozilla.org/en-US/docs/Web/API/ResizeObserver) for more details.
**waResizeObserver** will allow us to react when the component is resized

```javascript
<section>
    <h1 waResizeBox="content-box" (waResizeObserver)="onResize($event)">
        <div>
        width: {{this.width}}
        </div>
        <div>
        height: {{this.height}}
        </div>
        <span>
        domRectReadOnly: {{this.domRectReadOnly | json}}
        </span>
    </h1>
</section>
```
Creating `onResize` method in app.component.ts to assign resize event properties and display them in our template
```javascript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss']
})

export class AppComponent {
  title = 'resize-observer';
  width: number = 0;
  height: number = 0;
  domRectReadOnly: DOMRectReadOnly | undefined;

  onResize(entry: ResizeObserverEntry[]) {
    this.width = entry[0].contentRect.width
    this.height = entry[0].contentRect.height;
    this.domRectReadOnly = entry[0].contentRect;
  }
}
```
That's it. This is what happens when we resize the browser
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/8gvi80kmyufc545xn5o1.gif)

**Conclusions**
* We have reviewed what ResizeObserver is and how to implement it in Angular
* ResizeObserver is one of the API's for Angular, take a look into [this](https://ng-web-apis.github.io/) to find more

References
* [Web Apis for Angular](https://ng-web-apis.github.io/)
* [Web Apis for Angular repo](https://github.com/ng-web-apis)
* [ng-web-apis/resize-observer](https://github.com/ng-web-apis/resize-observer)
* [example repo](https://github.com/salimchemes/resize-observer)

---
title: 'How to implement ngrx-router-stores'
description: 'Implementing Router Store in NgRx'
pubDate: 'Jun 16 2020'
heroImage: '/ngrx-router-store.png'
---


Nowadays NgRx is a very popular framework mostly used when having an app with complex/shared state.
This is the list of packages offered by the framework today:
* Store: RxJS powered state management for Angular apps, inspired by Redux.
* Store Devtools: Instrumentation for @ngrx/store enabling time-travel debugging.
* Effects: Side effect model for @ngrx/store.
* **Router Store: Bindings to connect the Angular Router to @ngrx/store.**
* Entity: Entity State adapter for managing record collections.
* NgRx Data: Extension for simplified entity data management.
* NgRx Component: Extension for fully reactive, fully zone-less applications. 
* ComponentStore: Standalone library for managing local/component state.
* Schematics: Scaffolding library for Angular applications using NgRx libraries.

For more details you can check the [docs](https://ngrx.io/docs)

In this post, we will implement **Router Store**, step by step.

Why do we need **Router Store**? Basically to link the routing with the NgRx store. Every time the router changes, an action will be dispatched and will update the store through a reducer.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ktgzqvjevwfnpjmg8qmk.png)


We will divide the implementation in 4 steps, with an example of a list of movies and series:

**1. Add required dependencies**
**2. Update app.module.ts**
**3. Create router reducer and Custom Router State Serializer**
**4. Create a selector and subscribe from a component**

**1. Add required dependencies**
`npm install @ngrx/router-store --save`

**2. Update app.module.ts**
We need to
```javascript
 import { StoreRouterConnectingModule } from '@ngrx/router-store';
```

We import _StoreRouterConnectingModule_ to connect RouterModule with StoreModule, which has a serializer class named _CustomSerializer_, we will cover this in step #3

```javascript 
    StoreRouterConnectingModule.forRoot({
      serializer: CustomSerializer,
    }),
```
Assuming we have already implemented the _Store_ and _StoreDevtoolsModule_, this is how our app.module.ts looks
```javascript 
import { NgModule } from '@angular/core';
import { BrowserModule } from '@angular/platform-browser';
import { StoreRouterConnectingModule } from '@ngrx/router-store';
import { StoreModule } from '@ngrx/store';
import { StoreDevtoolsModule } from '@ngrx/store-devtools';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { MoviesDetailComponent } from './pages/movies-detail/movies-detail.component';
import { MoviesComponent } from './pages/movies/movies.component';
import { SeriesDetailComponent } from './pages/series-detail/series-detail.component';
import { SeriesComponent } from './pages/series/series.component';
import { CustomSerializer } from './store/custom-serializer';
import { reducers } from './store/index';

@NgModule({
  declarations: [
    AppComponent,
    MoviesComponent,
    SeriesComponent,
    SeriesDetailComponent,
    MoviesDetailComponent,
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    StoreModule.forRoot(reducers),
    StoreDevtoolsModule.instrument({
      maxAge: 25, // Retains last 25 states
      logOnly: true, // Restrict extension to log-only mode
    }),
    StoreRouterConnectingModule.forRoot({
      serializer: CustomSerializer,
    }),
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

**3. Create router reducer and Custom Router State Serializer**
Let's create _CustomSerializer_ class we set in app.module.ts, we want to just return some params and not the entire snapshot object to avoid possible performance issues
```javascript 
import { Params, RouterStateSnapshot } from '@angular/router';
import { RouterStateSerializer } from '@ngrx/router-store';

export interface RouterStateUrl {
  url: string;
  params: Params;
  queryParams: Params;
}

export class CustomSerializer implements RouterStateSerializer<RouterStateUrl> {
  serialize(routerState: RouterStateSnapshot): RouterStateUrl {
    let route = routerState.root;

    while (route.firstChild) {
      route = route.firstChild;
    }

    const {
      url,
      root: { queryParams },
    } = routerState;
    const { params } = route;

    // Only return an object including the URL, params and query params
    // instead of the entire snapshot
    return { url, params, queryParams };
  }
}
```
And finally we add our router reducer 
```javascript 
import { ActionReducerMap } from '@ngrx/store';
import * as fromRouter from '@ngrx/router-store';
import { routerReducer } from '@ngrx/router-store';

export interface StoreRootState {
  router: fromRouter.RouterReducerState<any>;
}
export const reducers: ActionReducerMap<StoreRootState> = {
  router: routerReducer,
};
```
**4. Create a selector and subscribe from a component**
We have it all set, the last step is to add a selector and subscribe to it from a component
Creating a selector

```javascript 
import * as fromRouter from '@ngrx/router-store';
import { createSelector } from '@ngrx/store';
import { StoreRootState } from '.';

export const getRouterState = (state: StoreRootState) => state.router;

export const getCurrentRouteState = createSelector(
  getRouterState,
  (state: fromRouter.RouterReducerState) => state.state
);

```
Subscribing from a component
```javascript
import { Component, OnDestroy, OnInit } from '@angular/core';
import { select, Store } from '@ngrx/store';
import { series } from 'src/app/app.constants';
import { StoreRootState } from 'src/app/store';
import { getCurrentRouteState } from 'src/app/store/selectors';

@Component({
  selector: 'app-series-detail',
  templateUrl: './series-detail.component.html',
  styleUrls: ['./series-detail.component.scss'],
})
export class SeriesDetailComponent implements OnInit, OnDestroy {
  seriesId: string;
  series;
  private subscriptions: { [key: string]: any } = {};

  constructor(private store: Store<StoreRootState>) {}

  ngOnInit(): void {
    this.subscriptions.routerSelector = this.store
      .pipe(select(getCurrentRouteState))
      .subscribe((route: any) => {
        const seriesId = route.params.seriesId;
        this.series = series.find((series) => series.id === seriesId);
      });
  }

  ngOnDestroy(): void {
    this.subscriptions.routerSelector.unsubscribe();
  }
}
```
The coding part is done, let's see how the example works

This is how the store looks when the app starts
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/k5h0vlgwkovnfqat9n67.png)

Let's navigate to the series list and see what happens in the store
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/1h0dyypi2obu14vqw363.png)

One more navigation to notice that route state has changed, including url and params
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/sv4s0nwdbyu2fz53es2t.png)

Thanks for reading!

References
* [ngrx website](https://ngrx.io/docs)
* [repo](https://github.com/salimchemes/ngrx-router-state)



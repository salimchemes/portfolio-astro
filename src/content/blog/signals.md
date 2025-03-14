---
title: 'Introducing Signals in Angular'
description: 'Introducing Signals in Angular'
pubDate: 'Mar 14 2025'
heroImage: '/signals.jpeg'

---

## What are signals?

Signals are a new reactive primitive that allows you to declare state dependencies explicitly. They automatically track dependencies and re-evaluate only when those dependencies change.

By using signals, Angular proposes a better reactivity approach to include

- fine grained control updates

- more scalable and better performance

**The goal of using signals is to make zone js optional in the future. Today there is an [experimental](https://angular.dev/guide/experimental/zoneless) to enable zoneless at application level**

Signals contain 3 primitives

- `signal`: A signal is a wrapper around a value that notifies interested consumers when that value changes. Signals can contain any value, from primitives to complex data structures.

```javascript
const count = signal(0); // needs a value when declared
```

- `computed`: read-only signals that derive their value from other signals. You define computed signals using the computed function and specifying a derivation:

```javascript
const count: Signal<number> = signal(0);

const doubleCount: Signal<number> = computed(() => count() * 2);
```

The doubleCount signal depends on the count signal. Whenever count updates, Angular knows that doubleCount needs to update as well.

- `effect`: Operation that runs whenever one or more signal values change. You can create an effect with the effect function:

```javascript
effect(() => {
  console.log(`The current count is: ${count()}`);
});
```

## Signals vs Observables (RxJS)

Signals and Observables can work together. This image illustrates where implement each.

![Screenshot 2025-02-18 at 11.23.33 AM.png](/signalsandrxjs.png)

- Close to the Template: It is better to use signals due to their superior change detection management and simpler reactivity. Signals provide fine-grained control over updates, making them ideal for managing state directly within components.

- Close to the Data Service: RxJS is preferred for manipulating streams and handling asynchronous reactivity. Observables are well-suited for complex asynchronous workflows, such as HTTP requests and real-time data streams.

This is not set in stone, Observables can be implemented on components and signals in services. It's just an approach to identify better what is the best place for each. **Especially useful when first implementing signals.**

## Signals in Components vs Signals in Services

use signals in components when

- retain unique value for each component instance
- communicate with parent component

use signals in services when

- share data
- retain data when component is destroyed

## Signals vs traditional inputs

We can use signal input instead of inputs

```javascript

// input

@Input() fetchCriteria: string;

// input signal

fetchCriteria = input<string>('');

```

One of the benefit of using input signals if we can create a computed signal to avoid using `ngOnChanges` when input gets a new value so we let the template react automatically without having to handle change detection manually

```javascript

// input

ngOnChanges(changes: SimpleChanges): void {

// adding any logic when input has a new value

}



// input signals

itemService = inject(ItemService);

items = this.itemService.items; // get a signal with all items

// no need of ngOnChanges and using a computed instead

filteredItems = computed(() =>

this.items().filter(s => s.name.includes(this.fetchCriteria())));

```

## Migrating to signals

Angular provides schematics to migrate to signals

- [inputs](https://angular.dev/reference/migrations/signal-inputs) `ng generate @angular/core:signal-input-migration`

tip: even migration would work with a signal without default value, is recommended to add it

```javascript
// TODO: Notes from signal input migration:

// Input is initialized to `undefined` but type does not allow this value.

// This worked with `@Input` because your project uses `--strictPropertyInitialization=false`.
```

![signals.gif](/.attachments/signals-8e24da01-264e-4b6c-a693-7b259dbc0f91.gif)

- [ouputs](https://angular.dev/reference/migrations/outputs) `ng generate @angular/core:output-migration`

- [queries](https://angular.dev/reference/migrations/signal-queries) `ng generate @angular/core:signal-queries-migration`

## Benefits of using signals

- one step forward to zoneless application

- better performance due to change detection is handled more efficiently avoiding not necessary extra rendering

- easier to understand than rxjs

## Drawbacks of using signals

- some features are still under developer preview so it is important to keep an eye on it before merging any pr

  - `linkedSignal`: https://angular.dev/guide/signals/linked-signal

  - `toObservable`: https://angular.dev/api/core/rxjs-interop/toObservable

- even support is growing, signals went out of developer preview on version 16, so there are still some things to improve in next versions. For example

  - signals forms https://github.com/angular/angular/issues/53485

- without a plan could lead to a mix of approaches between observables and signals

## References

- https://www.youtube.com/@deborah_kurata
- https://angular.dev/guide/signals
- https://www.youtube.com/@Angular
- https://angular.love/signal-store-ngxs-elevating-flexibility-in-state-management

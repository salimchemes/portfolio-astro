---
title: 'ReactiveForms, FormArrays and Custom Validators'
description: 'Implementing reactive forms, form arrays and custom validators in angular'
pubDate: 'Apr 25 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/vs7p47lw2fm0gpc7j92h.png'
---

In Angular we have 2 ways to work with Forms
* Template Driven: based on ngModel approach with 2 way data binding
* Reactive Forms: provide a model-driven approach to handling form inputs whose values change over time.

Template Driven is good when we don't have much complexity in our validations, but when we work with forms with complicated logic it's better to go with Reactive Forms because **we can implement the behavior we need in the component side and not in the template**. Adding validations just in the template is hard to understand and maintain.

In this post we will:
* Implement a ReactiveForm
* Add and remove FormArray items dynamically
* Implement custom validator functions 


Before we start with the coding I would like to recommend this 
[course](https://app.pluralsight.com/library/courses/angular-2-reactive-forms/table-of-contents) from Deborah Kurata, it helped me a lot to understand how RF works

First thing to do is add `ReactiveFormsModule` as part of our `app.module.ts`

```javascript
import { BrowserModule } from "@angular/platform-browser";
import { NgModule } from "@angular/core";

import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
import { ReactiveFormsModule } from "@angular/forms";

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, AppRoutingModule, ReactiveFormsModule],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Now we are ready to create our form using Reactive Forms. As part of our example, we will create an Author form which includes
* Add author name *(required and max length 40 characters)*
* Add books dynamically
  * book name *(required and max length 40 characters)*
  * stars *(required and 1 to 5)*

To perform validations Angular offers some built in validator functions. Those are:
```javascript
export declare class Validators {
    static min(min: number): ValidatorFn;
    static max(max: number): ValidatorFn;
    static required(control: AbstractControl): ValidationErrors | null;
    static requiredTrue(control: AbstractControl): ValidationErrors | null;
    static email(control: AbstractControl): ValidationErrors | null;
    static minLength(minLength: number): ValidatorFn;
    static maxLength(maxLength: number): ValidatorFn;
    static pattern(pattern: string | RegExp): ValidatorFn;
    static nullValidator(control: AbstractControl): ValidationErrors | null;
    static compose(validators: null): null;
    static compose(validators: (ValidatorFn | null | undefined)[]): ValidatorFn | null;
    static composeAsync(validators: (AsyncValidatorFn | null)[]): AsyncValidatorFn | null;
}
```
In case we need a validation that is not part of this list, we can create our own function, in the example we we will use both types, angular and custom validators.

Let's define the form structure using *FormBuilder*, a class to construct a new `FormGroup` instance. The form group has 2 properties, *author* (FormControl) and *books* (FormArray). Note that when declaring books, we use *FormBuilder* again to get a FormArray instance. We can set also default values if we want (check first author's array value). 
Finally we included a getter for our just created FormArray 


```javascript
import { Component, OnInit } from "@angular/core";
import { FormBuilder, FormGroup, Validators } from "@angular/forms";

@Component({
  selector: "app-root",
  templateUrl: "./app.component.html",
  styleUrls: ["./app.component.scss"],
})
export class AppComponent implements OnInit {
  title = "reactive-forms-and-form-arrays";
  myForm: FormGroup;

  get books(): FormArray {
    return this.myForm.get("books") as FormArray;
  }

  constructor(private fb: FormBuilder) { }

  ngOnInit() {
    this.myForm = this.fb.group({
      author: ["", [Validators.required, Validators.maxLength(40)]],
      books: this.fb.array([]),
    });
  }
}
```

As you can see, we already have defined *author* and *books* and also included 2 validators, **required** and **maxLength**


Now let's update our FormArray. We want to add and remove books dynamically. To do that, we add an item into the books to have one created   as default

```javascript

  private configForm() {
    this.myForm = this.fb.group({
      author: ["", [Validators.required, Validators.maxLength(40)]],
      books: this.fb.array([this.buildBook()]), //method to add 1 item by default
    });
  }

  private buildBook(): FormGroup {
    return this.fb.group({
      name: ["", [Validators.required, Validators.maxLength(40)]],
      stars: [null, [Validators.required, NumberValidators.range(1, 5)]],
    });
  }
```

Notice that *buildBook()* returns a new FormControl and it has 2 properties:
* name: required and max length 40 characters
* stars: required and with a range validator

We included a custom validator function to handle the stars FormControl, allowing 1-5 only. This is how the custom function looks
```javascript
import { AbstractControl, ValidatorFn } from '@angular/forms';

export class NumberValidators {

    static range(min: number, max: number): ValidatorFn {
        return (c: AbstractControl): { [key: string]: boolean } | null => {
            if ((c.value || c.value === 0) && (isNaN(c.value) || c.value < min || c.value > max)) {
                return { range: true };
            }
            return null;
        };
    }
}
```
Now let's add two methods, one to add a new book (using *buildBook()*)

```javascript
  addBook() {
    this.books.push(this.buildBook())
  }
```

and another to remove a specific book from the array

```javascript
 removeBook(i: number) {
    this.books.removeAt(i);
  }
```

We are ready to update our template. Fist we include the FormGroup and FormControlName *author* to match our component form definition

```javascript
<div [formGroup]="myForm" class="pt-5" style="width: 50%; margin:auto">
  <div>
    <h2>Author Form</h2>
    <h3 style="font-style: italic;">Reactive Forms, Form Arrays and Custom Validator functions</h3>
  </div>
  <div class="form-group">
    <label for="author">Author</label>
    <input type="text" class="form-control" placeholder="author name" formControlName="author" />
    <span *ngIf="myForm.get('author').errors?.required">required</span>
    <span *ngIf="myForm.get('author').errors?.maxlength">max 40 characters</span>
  </div>
</div>
```

There are two span elements to handle the errors defined, required and maxLength.

The last part is to integrate the FormArray into the template

```javascript
  <div class="form-group">
    <label for="exampleInputPassword1">Books</label>
    <div formArrayName="books">
      <div [formGroupName]="i" class="mt-3" *ngFor="let book of books.controls; let i=index">
        <div class="row">
          <div class="col-6">
            <input type="text" class="form-control" formControlName="name" placeholder="book name" />
            <span *ngIf="book.controls.name.errors?.required">required</span>
          </div>
          <div class="col-2">
            <input type="number" class="form-control" formControlName="stars" placeholder="book rate" />
            <span *ngIf="book.controls.stars.errors?.range">range 1 to 5</span>
            <span *ngIf="book.controls.stars.errors?.required">required</span>
          </div>
          <div class="col-1">
            <button class="btn btn-danger" (click)="removeBook(i)">X</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div>
    <button class="btn btn-primary" (click)="addBook()">add book</button>
  </div>
  <div>
    <button class="btn btn-primary mt-3" type="submit" [disabled]="!myForm.valid" (click)="save()">save</button>
  </div>
```
The most important to consider is the structure of the template
* formArrayName: name of the FormArray
* formGroupName: corresponds to a key in the parent FormArray
* formControlName: we have access to the controls of the iterated item, so we can use the formControlName we need

Finally, we add buttons to add, remove and save (only enabled if form is valid)

```javascript
<div [formGroup]="myForm" class="pt-5" style="width: 50%; margin:auto">
  <div>
    <h2>Author Form</h2>
    <h3 style="font-style: italic;">Reactive Forms, Form Arrays and Custom Validator functions</h3>
  </div>
  <div class="form-group">
    <label for="author">Author</label>
    <input type="text" class="form-control" placeholder="author name" formControlName="author" />
    <span *ngIf="myForm.get('author').errors?.required">required</span>
    <span *ngIf="myForm.get('author').errors?.maxlength">max 40 characters</span>
  </div>
  <div class="form-group">
    <label for="exampleInputPassword1">Books</label>
    <div formArrayName="books">
      <div [formGroupName]="i" class="mt-3" *ngFor="let book of books.controls; let i=index">
        <div class="row">
          <div class="col-6">
            <input type="text" class="form-control" formControlName="name" placeholder="book name" />
            <span *ngIf="book.controls.name.errors?.required">required</span>
          </div>
          <div class="col-2">
            <input type="number" class="form-control" formControlName="stars" placeholder="book rate" />
            <span *ngIf="book.controls.stars.errors?.range">range 1 to 5</span>
            <span *ngIf="book.controls.stars.errors?.required">required</span>
          </div>
          <div class="col-1">
            <button class="btn btn-danger" (click)="removeBook(i)">X</button>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div>
    <button class="btn btn-primary" (click)="addBook()">add book</button>
  </div>
  <div>
    <button class="btn btn-primary mt-3" type="submit" [disabled]="!myForm.valid" (click)="save()">save</button>
  </div>
  <div class="small">
    <br>author name errors: {{ myForm.get('author')?.errors | json }}
    <br>books [0] name errors: {{ books.get('0.name')?.errors | json }}
    <br>books [0] stars errors: {{ books.get('0.stars')?.errors | json }}
  </div>
</div>
```

Author validations
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/2yinrvm93h7ptemirshh.gif)

Books validations
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/zy9t60ipf9haskmhqp77.gif)

Add and remove items from books FormArray 
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/7hl8o4py5obg18w4avuu.gif)


references:
* [repo] (https://github.com/salimchemes/reactive-forms)
* [demo] (https://wizardly-haibt-e57583.netlify.app/)
* [course](https://app.pluralsight.com/library/courses/angular-2-reactive-forms/table-of-contents) from Deborah Kurata


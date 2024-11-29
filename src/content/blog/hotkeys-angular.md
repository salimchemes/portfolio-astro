---
title: 'Angular hotkeys (we all love shortcuts!)'
description: 'Angular hotkeys (we all love shortcuts!)'
pubDate: 'Dec 6 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/lmfl49zscrby4wa1nmh2.jpg'
---

Let's review how to implement hotkeys (shortcuts) on Angular apps.

**On this post, we will use a library called [angular2-hotkeys](https://github.com/brtnshrdr/angular2-hotkeys)**.

First, we will walk through the implementation and some extra configuration tips to finally go over some pros and cons as part of a conclusion.

**To understand better, we will create a simple app with**
* an input
* a save button with a shortcut command+s/control+s (mac/win)

**Installation**
`npm install angular2-hotkeys --save`

**_Important_**
If you get this [error](https://github.com/brtnshrdr/angular2-hotkeys/issues/136)
```javascript
ERROR in ../node_modules/angular2-hotkeys/src/hotkeys.service.d.ts:9:16 - error TS2304: Cannot find name 'MousetrapInstance'.
9     mousetrap: MousetrapInstance;
```
There is a [PR](https://github.com/brtnshrdr/angular2-hotkeys/pull/135) not merged yet with the fix.
Also, there is a workaround (it's on the repo used on this example) until the PR is merged and the new version is released: on devDependencies include
`"@types/mousetrap": "1.6.3"`



**Update app.module.ts**
```javascript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { BrowserModule } from '@angular/platform-browser';
import { HotkeyModule } from 'angular2-hotkeys';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';

@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    AppRoutingModule,
    CommonModule,
    ReactiveFormsModule,
    HotkeyModule.forRoot(), // adding HotkeysModule
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}

```

**Creating a component and adding a shortcut to save in the constructor**
```javascript
  constructor(private hotkeysService: HotkeysService) {
    this.hotkeysService.add(
      new Hotkey(
        'command+s', // key combination
        (): boolean => { // callback function to execute after key combination
          this.save(); 
          return false; // prevent bubbling
        },
        ['INPUT', 'TEXTAREA', 'SELECT'], // allow shortcut execution in these html elements
        'save' // shortcut name
      )
    );
  }
```

As we can see, **hotkeysService** allows us to add a new **HotKey** shortcut. Let's take a look into the class structure

```javascript
export declare class Hotkey {
    combo: string | string[];
    callback: (event: KeyboardEvent, combo: string) => ExtendedKeyboardEvent | boolean;
    allowIn?: string[];
    description?: string | Function;
    action?: string;
    persistent?: boolean;
    private formattedHotkey;
    static symbolize(combo: string): string;
    /**
     * Creates a new Hotkey for Mousetrap binding
     *
     * @param combo       mousetrap key binding
     * @param description description for the help menu
     * @param callback    method to call when key is pressed
     * @param action      the type of event to listen for (for mousetrap)
     * @param allowIn     an array of tag names to allow this combo in ('INPUT', 'SELECT', and/or 'TEXTAREA')
     * @param persistent  if true, the binding is preserved upon route changes
     */
    constructor(combo: string | string[], callback: (event: KeyboardEvent, combo: string) => ExtendedKeyboardEvent | boolean, allowIn?: string[], description?: string | Function, action?: string, persistent?: boolean);
    get formatted(): string[];
}
```

We included **allowInput with INPUT** in order to get our combo working even when we have focus on it. If we don't add it, since command+s is a browser shortcut to save the page, our shortcut will not be executed and instead, the save file prompt will appear (you can try removing allowIn parameter and trying to run the combo)

Now we will add a formGroup and some logic to support 
* _command+s_ on Mac
* _control+s_ on Windows

```javascript
import { Component, OnInit } from '@angular/core';
import { FormBuilder, FormGroup } from '@angular/forms';
import { Hotkey, HotkeysService } from 'angular2-hotkeys';

@Component({
  selector: 'app-my-form-with-hot-keys',
  templateUrl: './my-form-with-hot-keys.component.html',
  styleUrls: ['./my-form-with-hot-keys.component.scss'],
})
export class MyFormWithHotKeysComponent implements OnInit {
  mac = 'command+s';
  win = 'ctrl+s';
  isMac = navigator.platform.includes('Mac');
  saveCommand = this.isMac ? this.mac : this.win;
  saveCommandTitle = this.isMac ? '⌘+s' : this.win;
  form: FormGroup;
  constructor(private hotkeysService: HotkeysService, private fb: FormBuilder) {
    this.hotkeysService.add(
      new Hotkey(
        this.saveCommand, //  key combination
        (): boolean => {
          // callback function to execute after key combination
          this.save();
          return false; // prevent bubbling
        },
        ['INPUT', 'TEXTAREA', 'SELECT'], // allow shortcut execution in these html elements
        'save' // shortcut name
      )
    );
  }

  ngOnInit(): void {
    this.configForm();
  }

  save() {
    alert(this.form.controls?.textValue?.value || 'no text');
  }

  private configForm() {
    this.form = this.fb.group({
      textValue: '',
    });
  }
}
```
adding the template
```javascript
<div class="form" [formGroup]="form">
  <div class="title">hot keys</div>
  <div>
    <label for="textValue" class="text-label">Type some text</label>
    <input
      type="text"
      id="textValue"
      formControlName="textValue"
      class="input-form"
    />
  </div>
  <div>
    <button (click)="save()" title="{{ saveCommandTitle }}">save</button>
  </div>
</div>

```

Let's see how it looks so far
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/mqxtbwuly7xz519rczj1.gif)

Nice! it's working, we can save clicking the save button or running the shortcut!

**Cheatsheet**
The library supports a cheatsheet with a list of all shortcuts registered in our app, the only thing we need to do is add
```javascript
<hotkeys-cheatsheet></hotkeys-cheatsheet>
```

We will add it in app.component.html (we can pass a custom title)
```javascript
<hotkeys-cheatsheet title="Hotkeys map"></hotkeys-cheatsheet>
<app-my-form-with-hot-keys></app-my-form-with-hot-keys>
```
By default, to see the cheatsheet, we need to press *?*. We will override default values to see what else we can setup. You can check the comments added on each property.

For example, instead of using _?_ to show the cheatsheet, we will use a _!_

In app.module.ts, we will pass options using the interface **IHotkeyOptions**.

**Important: Only shortcuts with description will appear in the cheatsheet. Look at [this](https://github.com/brtnshrdr/angular2-hotkeys/blob/2014f38facafcc77ac2e34ea6b579227cc57d51b/src/lib/hotkeys-cheatsheet/hotkeys-cheatsheet.component.ts#L24) to see how the component works  inside the library**

             

```javascript
import { CommonModule } from '@angular/common';
import { NgModule } from '@angular/core';
import { ReactiveFormsModule } from '@angular/forms';
import { BrowserModule } from '@angular/platform-browser';
import { HotkeyModule, IHotkeyOptions } from 'angular2-hotkeys';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { MyFormWithHotKeysComponent } from './components/my-form-with-hot-keys/my-form-with-hot-keys.component';

const options: IHotkeyOptions = {
  disableCheatSheet: false, // disable the cheat sheet popover dialog? Default: false
  cheatSheetHotkey: '!', // key combination to trigger the cheat sheet. Default: '?'
  cheatSheetCloseEsc: true, // use also ESC for closing the cheat sheet. Default: false
  cheatSheetCloseEscDescription: 'hide hotkeys map', // description for the ESC key for closing the cheat sheet (if enabed). Default: 'Hide this help menu'
  cheatSheetDescription: 'show all hotkeys', // description for the cheat sheet hot key in the cheat sheet. Default: 'Show / hide this help menu'
};

@NgModule({
  declarations: [AppComponent, MyFormWithHotKeysComponent],
  imports: [
    BrowserModule,
    AppRoutingModule,
    CommonModule,
    ReactiveFormsModule,
    HotkeyModule.forRoot(options), // adding options instance when registering module
  ],
  providers: [],
  bootstrap: [AppComponent],
})
export class AppModule {}
```
Finally, typing _!_ the cheatsheet is displayed

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/dddpey9xjyhe1dpche53.gif)

Conclusions
Pros
* The library is easy to implement and offers some handy features
* Cheatsheet is cool!

Cons
* Last release was on march this year and even there are some PRs with important fixes including angular 8+ support the release is still pending

Thanks for reading! If you like this post, give it a 🦄 ❤️ 🔖.

References
* [demo](https://sharp-cray-f033e1.netlify.app/)
* [repo](https://github.com/salimchemes/angular-hotkeys)
* [angular2-hotkeys](https://github.com/brtnshrdr/angular2-hotkeys)
* [npm](https://www.npmjs.com/package/angular2-hotkeys)
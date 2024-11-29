---
title: 'Generating text description from youtube link using Angular, Gemini and Netlify'
description: 'Generating text description from youtube link using Angular, Gemini and Netlify'
pubDate: 'Sept 25 2024'
heroImage: 'https://media2.dev.to/dynamic/image/width=1000,height=420,fit=cover,gravity=auto,format=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fveg1kaptgplwrczolq36.jpg'
---
In this post we will implement an application to generate ai description from a youtube link using:
* angular with primeng for the ui
* gemini to generate the description
* netlify for ci/cd

## Create app using angular cli

```ng new angular-youtube-description```

## Add required dependencies

**gemini**
* `npm i @google/generative-ai`

**primeng**
* `npm install primeng`
* in angular.json add
```
...
"styles": [
    "node_modules/primeng/resources/themes/lara-light-blue/theme.css",
    "node_modules/primeng/resources/primeng.min.css",
    ...
]
```
and in styles.css add
```
@import "primeng/resources/themes/lara-light-blue/theme.css";
@import "primeng/resources/primeng.css";
```

## Adding initial changes on app.component.ts
Let's start adding some changes into `app.component.ts`. Each property has a comment with an explanation

```javascript
import { CommonModule } from '@angular/common';
import { Component, inject, OnInit, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { GoogleGenerativeAI } from '@google/generative-ai';
import { FormsModule } from '@angular/forms';
import { HttpClient, HttpClientModule } from '@angular/common/http';
import { take } from 'rxjs';
import { ButtonModule } from 'primeng/button';
import { InputTextModule } from 'primeng/inputtext';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    RouterOutlet,
    CommonModule,
    FormsModule,
    HttpClientModule,
    ButtonModule,
    InputTextModule
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent implements OnInit {
  generatedDescription = signal<string | undefined>(undefined); // to see the description response in the template
  videoUrl = signal<string | undefined>(undefined); // url we want to  send to gemini
  error = signal<string | undefined>(undefined); // to display an error in template when failing
  isLoading = signal<boolean | undefined>(undefined); // to display in template when waiting for gemini response
  #genAI: GoogleGenerativeAI | undefined; // to build a gemini model and generate content from prompt
  #youtubeApiKey: string | undefined = ''; // add youtube key required to get video data
  #geminiApiKey: string | undefined = ''; // add gemini key required to get gemini response
  #http = inject(HttpClient); // to make a request to youtube video so we can get title and description to send to gemini

  ngOnInit() {
    this.#genAI = new GoogleGenerativeAI(this.#geminiApiKey!);
  }
```

## Generating required key for gemini

1. go to [Google AI Studio](https://aistudio.google.com/app/prompts/new_chat).
2. login with your Google account.
3. create an [API key](https://aistudio.google.com/app/apikey).
4. once you get the key, assign it to ```#geminiApiKey``` in `app.component.ts`

## Generating required key for youtube
1. follow this instructions https://developers.google.com/youtube/v3/getting-started
2. once you get the key, assign it to ```#youtubeApiKey``` in `app.component.ts`


## Adding logic to get youtube video data and gemini response
 `getVideoDetails(videoUrl: string | undefined)`: we make a request to get youtube title and description so we can create gemini prompt. Since we need a videoId we added `extractVideoId(url: string)` to extract it from url

`generateDescription(title: string, originalDescription: string)`: in this method we create prompt to request gemini. We need to specify a model as parameter, in this case we use 'gemini-pro'. Finally we execute `generateContent` to assign the response to `generatedDescription` signal so we can display it in the template
```javascript
import { CommonModule } from '@angular/common';
import { Component, inject, OnInit, signal } from '@angular/core';
import { RouterOutlet } from '@angular/router';
import { GoogleGenerativeAI } from '@google/generative-ai';
import { FormsModule } from '@angular/forms';
import { HttpClient, HttpClientModule } from '@angular/common/http';
import { take } from 'rxjs';
import { ButtonModule } from 'primeng/button';
import { InputTextModule } from 'primeng/inputtext';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [
    RouterOutlet,
    CommonModule,
    FormsModule,
    HttpClientModule,
    ButtonModule,
    InputTextModule
  ],
  templateUrl: './app.component.html',
  styleUrl: './app.component.css',
})
export class AppComponent implements OnInit {
  generatedDescription = signal<string | undefined>(undefined);
  videoUrl = signal<string | undefined>(undefined);
  error = signal<string | undefined>(undefined);
  isLoading = signal<boolean | undefined>(undefined);
  #genAI: GoogleGenerativeAI | undefined;
  #youtubeApiKey: string | undefined = ''; // add youtube key required to get video data
  #geminiApiKey: string | undefined = ''; // add gemini key required to get gemini response);
  #http = inject(HttpClient);

  ngOnInit() {
    this.#genAI = new GoogleGenerativeAI(this.#geminiApiKey!);
  }

  getVideoDetails(videoUrl: string | undefined): void {
    this.error.set(undefined);
    const videoId = this.extractVideoId(videoUrl!);
    if (!videoId) {
      this.error.set('Invalid YouTube URL');
      return;
    }
    const url = `https://www.googleapis.com/youtube/v3/videos?part=snippet,contentDetails,statistics&id=${videoId}&key=${
      this.#youtubeApiKey
    }`;
    this.isLoading.set(true);
    this.#http
      .get(url)
      .pipe(take(1))
      .subscribe({
        next: (response: any) => {
          const videoData = response.items[0];
          this.generateDescription(
            videoData.snippet.title,
            videoData.snippet.description
          );
        },
        error: () => {
          this.isLoading.set(false);
          this.error.set('Unable to fetch video details');
        },
      });
  }

  private generateDescription(
    title: string,
    originalDescription: string
  ): void {
    const prompt = {
      input: `Generate a description for a YouTube video with the title: "${title}" and the following description: "${originalDescription}"`,
    };
    this.#genAI
      ?.getGenerativeModel({ model: 'gemini-pro' })
      .generateContent(prompt.input)
      .then((response: any) => {
        this.generatedDescription.set(
          response.response.candidates[0].content.parts[0]?.text
        );
        this.isLoading.set(false);
      })
      .catch(() => {
        this.isLoading.set(false);
        this.error.set('Unable to get response from Gemini AI model');
      });
  }

  private extractVideoId(url: string): string | null {
    const regex =
      /(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/(?:[^\/\n\s]+\/\S+\/|(?:v|e(?:mbed)?)\/|\S*?[?&]v=)|youtu\.be\/)([a-zA-Z0-9_-]{11})/;
    const match = url.match(regex);
    return match ? match[1] : null;
  }
}

```

## Updating template on app.component.html

```javascript
<div class="app-container">
  <div class="app">
    <h1>Enter youtube video url</h1>
    <input
      type="text"
      pInputText
      [(ngModel)]="videoUrl"
      placeholder="Enter youtube video url"
    />
    <p-button
      type="submit"
      [disabled]="isLoading()"
      (click)="getVideoDetails(videoUrl())"
    >
      Get description
    </p-button>
    @if(isLoading()){
    <p class="loading">Generating description from youtube video...</p>
    } @else if(generatedDescription()){
    <p>{{ generatedDescription() }}</p>
    <p><i>Generated description by Gemini</i></p>
    } @if(error()){
    <p class="error">{{ error() }}</p>
    }
  </div>
</div>

```

## Updating styles on app.component.css
```javascript
.app-container {
  justify-content: center;
  align-items: center;
}

.app {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-inline: 20%;
}

p-button {
  margin-top: 20px;
}

input {
  width: 350px;
}

.error {
  color: red;
}

.loading {
  font-style: italic;
}

```
## Deploying app using Netlify
We don't want to expose our keys in our code, so we will use `@ngx-env/builder` to add our keys in a `.env` file to finally create environment variables on Netlify portal
1 - `npm i @ngx-env/builder` and follow [documentation]( https://www.npmjs.com/package/@ngx-env/builder) 
2 - update `env.d.ts`
```javascript
declare interface Env {
  readonly NODE_ENV: string;
  // Replace the following with your own environment variables.
  NG_APP_YOUTUBE_API_KEY: string;
  NG_APP_GEMINI_API_KEY: string;
  [key: string]: any;
}
```
2 - add your key to `.env` file (IGNORE IT - DO NOT PUSH IT)
```
NG_APP_YOUTUBE_API_KEY=add your youtube api key here
NG_APP_GEMINI_API_KEY=add your gemini api key here
```
3 - update `app.component.ts` to use env variables
```javascript
#youtubeApiKey: string | undefined = import.meta.env.NG_APP_YOUTUBE_API_KEY;
#geminiApiKey: string | undefined = import.meta.env.NG_APP_GEMINI_API_KEY;
```


4 - deploy to Netlify following [these steps](https://dev.to/salimchemes/deploying-angular-app-with-netlify-in-3-steps-55k6)

5 - add environment variables in Netlify 
Go to site configuration

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/e8dtbz3bx26z6xofeney.png)


![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xg4dgpt3qskv2tcm489d.png)

6 - deploy your website

And that is it! Thanks for reading!

links:

- deployed website: https://unrivaled-choux-3b74f5.netlify.app/
- repo: https://github.com/salimchemes/angular-gemini
- gemini: https://gemini.google.com/
- @ngx-env/builder: https://www.npmjs.com/package/@ngx-env/builder
- primeng: https://primeng.org/









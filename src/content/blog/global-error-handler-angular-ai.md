---
title: 'Global Error Handler in Angular + Application Insights'
description: 'How to implement a global error handler'
pubDate: 'Apr 19 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/ruoumj2zf8bt2l2tgf2l.png'
---
 
We don't like errors, but they will happen anyway, so it's important to have a centralized way to handle errors in our angular app. We want to catch, identify and take actions with them.
In this post we will:
* Implement global error handler in angular
* Add Application Insights (aka AI) sdk
* Track errors in AI


**Implement global error handler in angular**
Angular makes our life easier catching the errors globally thank to <a href="https://angular.io/api/core/ErrorHandler" target="_blank">ErrorHandler</a> class, so let's see how to implement it
* Create Global Error Handler service and implement ErrorHandler class
```javascript
import { Injectable, ErrorHandler } from "@angular/core";
import { HttpErrorResponse } from "@angular/common/http";

@Injectable({
  providedIn: "root",
})
export class GlobalErrorHandler implements ErrorHandler {
  constructor() {}

  handleError(error: Error | HttpErrorResponse) {}
}
```
* Update app.module.ts providers
```javascript
import { BrowserModule } from "@angular/platform-browser";
import { NgModule, ErrorHandler } from "@angular/core"; 
import { AppRoutingModule } from "./app-routing.module";
import { AppComponent } from "./app.component";
import { HttpClientModule } from "@angular/common/http";
import { GlobalErrorHandler } from "./services/global-error-handler";

@NgModule({
  declarations: [AppComponent],
  imports: [BrowserModule, HttpClientModule, AppRoutingModule],
  providers: [{ provide: ErrorHandler, useClass: GlobalErrorHandler }], // Our service added

  bootstrap: [AppComponent],
})
export class AppModule {}
```

**Add Application Insights sdk**
We need to add this [dependency](https://github.com/microsoft/applicationinsights-js) in our app
* `npm i --save @microsoft/applicationinsights-web`

Now let's create a service to send exceptions to AI

```javascript
import { Injectable } from "@angular/core";
import { ApplicationInsights } from "@microsoft/applicationinsights-web";
import { environment } from "src/environments/environment";
@Injectable({
  providedIn: "root",
})
export class ApplicationInsightService {
  appInsights: ApplicationInsights;
  constructor() {
    this.appInsights = new ApplicationInsights({
      config: {
        connectionString: environment.appInsightsConfig.connectionString, // provided by Azure
        /* ...Other Configuration Options... */
      },
    });
    this.appInsights.loadAppInsights();
    this.appInsights.trackPageView();
  }

  logError(error: Error) {
    this.appInsights.trackException({ exception: error });
  }
}

```




And then integrate it with our global error handler service
```javascript
import { Injectable, ErrorHandler } from "@angular/core";
import { HttpErrorResponse } from "@angular/common/http";
import { ErrorService } from "./error.service";
import { LogService } from "./log.service";

@Injectable({
  providedIn: "root",
})
export class GlobalErrorHandler implements ErrorHandler {
  constructor(
    private errorService: ErrorService,
    private logService: LogService
  ) {}

  handleError(error: Error | HttpErrorResponse) {
    if (error instanceof HttpErrorResponse) {
      // Server error
      alert(this.errorService.getHttpError(error));
    } else {
      // Client Error
      this.logService.logErrorOnApplicationInsight(error);
      alert(this.errorService.getClientSideError(error));
    }
    // Always log errors
    this.logService.logError(error);
  }
}
```
logService is just a wrapper to take log actions

```javascript
import { Injectable } from "@angular/core";
import { ApplicationInsightService } from "./application-insight.service";
import { HttpErrorResponse } from "@angular/common/http";

@Injectable({
  providedIn: "root"
})
export class LogService {
  constructor(private applicationInsightService: ApplicationInsightService) {}

  logErrorOnApplicationInsight(error: Error) {
    return this.applicationInsightService.logError(error);
  }

  logError(error: Error | HttpErrorResponse) {
    console.error(error);
  }
}

```
**Track errors in AI**
To be able to see errors we send from the app we need to
* Create AI artifact (need a resource group first)
* Get connection string and add it into our app (you will find it in Azure portal)
* Throw an error from our app and track exception (check [example](https://github.com/salimchemes/global-error-handler) app)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/aed7195qzk9n8omlpgo6.gif)

This is how the errors look in AI
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/ypxn69ed2hru8tttmpwm.png)

**References**
* [github repo](https://github.com/salimchemes/global-error-handler)
* [AI sdk](https://github.com/microsoft/applicationinsights-js) 
* [AI docs](https://docs.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

---
title: 'e2e experience with cypress.io'
description: 'implementing automation testing on complex ui scenarios'
pubDate: 'Mar 14 2019'
heroImage: 'https://www.cypress.io/cypress_logo_social.png'
---

I used to work on a project where we had to implement automation tests.

Due to the complexity of our validation scenarios (not easy UI), our releases became very fragile and we got some production issues.

To fix that, our team created an Automation Test plan. Basically we made a list with the most critical scenarios to cover.

Once we had the list ready, it was time to discuss about which technology use. We decided to try [cypress.io](https://www.cypress.io/)


Cypress is a javascript end to end framework testing.
It has its own architecture not using Selenium and it has a wrapper of some existing technologies that we normally use independently.
Behind cypress is a Node.js server process. Cypress and the Node.js process constantly communicate, synchronize, and perform tasks on behalf of each other.

This image is from cypress website and explains very well how it works.

 ![Cypress how it works](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%208.01.53%20PM.png?alt=media&token=63946648-fd7e-429e-aa49-4ec81f61179a)


So, back to our story, we started with cypress, we knew it does not include IE support. Nevertheless, we gave it a try.

Cypress tests are written in javascript and also has jquery selectors to identify DOM elements easily. It means that having web developer skills is enough to write the tests.

Since its based on promises, you don't have to take care about waits, delays or any kind of manual implementations, all you need will be there with the promise callbacks (in most cases handled internally on the methods)

Also, has a very cool electron app to run the tests. And the documentation is one of the best parts, very well explained with great examples.

So finally, we wrote all our test defined on the test plan and then included them into our pipeline (Azure). Tests are very stable and we also were able to prevent some production issues thank to them. We trust in our tests.

So, let's take a look into a examples provided from cypress scaffolding 
First thing we need to do is **(make sure you have a package.json file in your project path)**:

    cd /your/project/path

    npm i cypress --save-dev

    ./node_modules/.bin/cypress open

 ![Cypress how it runs](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%206.47.42%20PM.png?alt=media&token=c3e30342-a165-477e-8dac-d5505840a3ec)


The electron app will start and you will see all the tests that are part of the examples provided by the cypress scaffolding:

![Cypress Scaffolding](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%206.50.55%20PM.png?alt=media&token=2ff71c09-a6ed-48f6-9520-ee26e4d34d29)

You can select if you want to run your tests on Chrome, Chromium or Electron
![Cypress test runner](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%206.52.49%20PM.png?alt=media&token=7012fcce-c670-4c31-b0ab-0c15091db6ca)

Once you click any test, you will see logs on the left side and the tested web site at the right. If something fails you will be able to review the logs to identify and fix the issue into the code.
You can run all the test clicking on Run all specs button. Also you can run as headless and will be able to get a video as test result (you can configure it on cypress.json file)

![Cypress runner 2](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%207.05.38%20PM.png?alt=media&token=7a06b957-92fc-4a31-bf51-8653b337c1c6)

The framework is flexible and you can do different implementations like create your own cypress commands (there is a command.js file under support folder to do that) or change the viewport size. Take a look into the tests provided as example and you will find several scenarios.

Now, let's take a look to the tests. These are the provided examples. If you are familiar with frameworks like jasmine, mocha, chai, etc, you will be able to write tests quickly.
You can review the examples and play around with them.

![Cypress test code](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%2010.45.46%20PM.png?alt=media&token=d99a92ef-8e96-4c31-8c36-d9ed854182ba)

Once you have your tests written, you can run them in different ways with the command **cypress run** (headless, browser, etc). Check [here](https://docs.cypress.io/guides/guides/module-api.html#cypress-run) to see the options




**CI Integration**: We have implemented our test running on Azure DevOps (ex VSTS), you will find several examples [here](https://docs.cypress.io/guides/guides/continuous-integration.html#Examples) to find what you need

Lets recap Cypress experience

**The good part** 
* based on 100% on js and promises
* performance is very good
* easy to learn and implement for web developers
* documentation is clear
* very stable
* cool electron app to run the tests and dashboard

**The not too good part**
* just support chrome, chromium, canary and electron (they are going to fix that probably [check this github issue for ie](https://github.com/cypress-io/cypress/issues/1564)). **If you need to support other browser this is important to know before start** 
 
Also, cypress is becoming very popular today. This is part of the state of js [survey](https://2018.stateofjs.com/testing/other-libraries/) from last year (2018, 20k devs participated)

![Cypress stateojs 1](https://firebasestorage.googleapis.com/v0/b/lawyer-f1b6d.appspot.com/o/Screen%20Shot%202019-03-10%20at%207.37.44%20PM.png?alt=media&token=3c7f98ec-5732-4434-8132-843e53caee49)


And this is part of the [conclusion](https://2018.stateofjs.com/testing/conclusion/ )

_“The future of testing may include more solutions to make automated tests in the browser, a project like Cypress may be included in the next year survey and we may see more tools based on Puppeteer.”_

**Conclusion**: 
After implementing cypress, the results were positive. We feel we can trust our tests and also we have a better coverage for every change we add. 
If you have to start an e2e test implementation, I would suggest give a try to cypress. All the needs have different aspect to analyze to make decisions, so at least keep in mind there are other options to consider.

References [cypress website](https://cypress.io)
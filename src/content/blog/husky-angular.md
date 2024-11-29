---
title: 'Husky hooks in Angular 🐶'
description: 'Husky hooks in Angular 🐶'
pubDate: 'Nov 23 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/pokhj1xc8rlunrrfri56.jpg'
---

**What is husky?**
Husky prevents push/commit changes to our repo that are not desired like tests failing or not well formatted files. If we try to commit something that is not correct, 🐶 will say: Woof!

**How it works?**
* `npm install husky --save-dev`
* adds your hooks into package.json
```javascript
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "npm test",
      "pre-push": "npm test",
      "...": "..."
    }
  }
}
```

**How husky can help us and what are we going to cover on this post?**
* Run Prettier and avoid not well formatted files
* Run tests and make sure all of them pass before pushing

**What is Prettier?**
Prettier formats our code in order to have an unified pattern for the project files. 

**How it Works?**
You need to install the dependency and follow the next steps:
* `npm i prettier --save-dev`
* adds .prettierrc to let your editor know that you are using Prettier
* adds .prettierignore to exclude files to be formatted

We have Prettier running in our project so let's adds our first hook

**Hook #1: Prettier**
First of all we need to install
* `npm install --save-dev pretty-quick husky`
* adds a pre-commit hook on package.json
```javascript
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "pretty-quick --staged"
    }
  }
}
```
That's all, now we are going to see if it works

I will add some extra spaces in a app.component.html (could be any file)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/0zwo1io8fhynzfxtb1rx.gif)

Finally I will commit to see what happens

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/d1aepap83p1nrf3xsb2u.png)

Nice! Prettier pre-hook fixed my file (no extra spaces anymore)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/tgicke2qzz1o52up109e.png)

You can also use Prettier Visual Code [extension](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode) and setup the IDE to format the code when saving files. But it will work just locally and you will need this configuration in all developer computers from your team. If for some reason any developer does not have this setup, some not formatted code could be pushed generating posible merge conflicts.


**Hook #2: Running tests**
This one is very simple, we will prevent commits with unit test failing. The only thing we need to do is add an extra sentence in our pre-commit hook (note that we are running the test headless to don't open any browser)
```javascript
// package.json
{
  "husky": {
    "hooks": {
      "pre-commit": "pretty-quick --staged && ng test --watch=false --browsers=ChromeHeadless
    }
  }
}
```
Let's make a test fail and try to commit

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/lw27hsbh4obpc3w0yklq.png)

Since there is a failing test, I am not able to commit.
Let's fix the test and try again.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/cupsk6bsmy78lgyu655k.png)

**Conclusions**
* Husky hooks are helpful to have a standard way to commit/push code
* Prettier will format the code for us avoiding merge conflicts and helping us to get our project files more clean and organised
* You can add as many hooks as you want, in this post we just reviewed 2 but could be more

References
* [husky] (https://www.npmjs.com/package/husky)
* [husky hooks] (https://typicode.github.io/husky/#/)
* [prettier] (https://prettier.io/docs/en/index.html)
* [prettier and pre commit hooks] (https://prettier.io/docs/en/precommit.html)
* [repo](https://github.com/salimchemes/husky-prettier)


Thanks for reading!
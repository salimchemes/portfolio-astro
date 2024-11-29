---
title: 'Deploying Angular app with Netlify in 3 steps'
description: 'Deploying Angular app with Netlify in 3 steps'
pubDate: 'May 10 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/6syudn32kwrpwmuzobfl.png'
---

Netlify is a great platform to build/deploy any kind of web applications (not just Angular).
It's very useful when you need to deploy your app fast and easy.

I found it handy to have demos or examples running when writing posts or when I need to have some coding working and live (not just local), but Netlify is not just for that, it is a very powerful platform.

We can deploy our Angular app following these steps:
  **1. Create your angular project on github (could also be on bitbucket/gitlab)**
  **2. Log in into Netlify, look for your repo and setup the build options**
  **3. Deploy the new web site created!**

Let's start

**1. Create you angular project on github (could also be on bitbucket/gitlab)**
```javascript
ng new my-angular-app
```


Create a repo on github and push your code.


**2. Log in into Netlify, look for your repo and setup the build options**
* Log in https://www.netlify.com/ 
* Clic on New site from Git
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/y937ysi511vwzolxaoyj.png)
* Select Github as provider
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/6ih45xbfwg8qpnknlvpl.png)
* After authorization, we will see the list of available repositories to pick. 
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/b9ih5m4ybfvggeig9my3.png)

If `my-angular-app` repo is not on the list, we need to provide access from github. (If you see your repo, you can skip this step). 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/a2yohsguf2fbqgij1ywv.png)
Clic on the highlighted link "_Configure the Netlify app on GitHub_". 
We will be redirected to github to look for our missing repository

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/hy0qkojtqtjfy4lqb0gn.png)

* Now we can see `my-angular-app`
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/g5sg2r7vfr1f2rj0racd.png)

* As part of build options setup, this is what we need:
  1. build command: we build our code in prod mode
  2. publish directory: location of build files
**(UPDATE: add /browser to publish directory)**
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/jir74s88qzq0851oet7a.png)

**3. Deploy the new web site created**

After clic on Deploy site, the first build is triggered and deploy is in progress
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/nrysy1h1d3r8uz50sxuy.png)

Finally, we have our site running
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/87n75wf1arlx8kgy2w9c.png)

Let's go to the site list to see one we have just created
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/o5zckr6nae2mp0lrs8cd.png)

Clic on our site, and then on the url provided by Netlify
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/tee3khqmcxp20f2jmbvw.png)

That's all! site deployed and running! 

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/n67ctdyexaac1407rn1n.png)

**Conclusions**

Netlify provides a lot of cool features and tools, this post is just to demo how to deploy fast with Angular but there is a lot more to work with.
Other Netlify features
* Custom domains
* Functions with AWS Lambda
* Identity
* Forms
* Large Media
* Split Testing
* Analytics

**References**
* github [repo](https://github.com/salimchemes/my-angular-app) (nothing special here)
* Netlify site running: https://focused-bhaskara-dee416.netlify.app/

---
title: 'Analyzing Angular bundle with Webpack Bundle Analyzer'
description: 'Implementing reactive forms, form arrays and custom validators in angular'
pubDate: 'Oct 5 2020'
heroImage: 'https://dev-to-uploads.s3.amazonaws.com/i/lvgwywdaw2rww0qmq8cu.png'
---

Angular, and every modern web application, includes dependencies to makes our life easier. Even most of them are optimised, this is not free at all. Every dependency we add is going to increase our bundle. 

We don't want to re-invent the wheel, but I think before adding more stuff to our bundle we should ask before:
* Is this exactly what I need?
* How is going to impact the bundle?

Another important thing to consider is that we should implement lazy-loading as much as we can in order to split our code better and improve the performance. You can take a look to this [post](https://dev.to/salimchemes/optimising-angular-performance-with-lazy-loading-and-quicklink-3j66) for more details.

**So, what is this post about? We will review webpack-bundle-analyzer, a tool to see what is inside of our bundle. It will create an interactive treemap visualization of the contents of all your bundles. We can navigate the map to identify what can be removed/optimized**

Steps to install
```javascript
 npm install --save-dev webpack-bundle-analyzer
 
 ng build --stats-json

 npx webpack-bundle-analyzer dist/your-project-name/stats.json  
```

Note that after dist/ you need to set your project name, I am using an example project including some extra dependencies to be reviewed on the map. In a real world app, you will probably have more dependencies and modules to analyze

After running the last command line you will have this
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/4ojp5ikhoi4teld39t2m.png)

This is how the map looks
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/jmc3g6s5vpzlf4jqzif1.gif)

Now let's navigate moment js, as you can see, moment is not a small dependency
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/zhufyrq35gju5y2z2nff.gif)

Finally we can look for a specific module
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/xtxo78qmeop1fp8rc95q.gif)

Conclusions
webpack-bundle-analyzer is a very useful tool to
* Get a big picture of our project
* Identify dependencies that can be replaced with small ones
* Identify big modules and split them in smaller pieces

References
* [webpack-bundle-analyzer] (https://www.npmjs.com/package/webpack-bundle-analyzer)



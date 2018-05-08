---
layout: post
title: One package.json for multiple apps
date: '2018-04-15'
author: Frans Lytzen
tags: javascript, typescript
modified_time: '2018-04-15''
excerpt: When you have multiple javascript apps in a single solution, you can reduce build times and version conflicts by having a shared package.json used by all the projects. This post walks you through the subtleties of setting this up, using Webpack and Typescript.
---
# The scenario
We are increasinly building systems where we have multiple web apps for different audiences. We combine them all in a single solution and use Docker to spin up all the individual micro-sites for each audience. One thing that has been a bone of contention it that each javascript SPA would have it's own `package.json` file. This has several disadvantages, incuding;
- You have to manually keep versions in sync between the different `package.json` files.
- You can't (currently) use things like [Green Keeper](TBD) to keep an eye on your versions.
- Build times are longer because you have to run `npm install` for each app, downloading mostly the same dependencies. This is especially troublesome on build servers that don't cache between each build.

**I am using Typescript and Webpack. Both the problems and solutions are specific to this combination so if you are not using these, this post is probably not for you**. I am also using React in this example, but that doesn't matter at all, it's just what I usually use. I also use `SASS` in the example but you should easily be able to subsitute that for something else.
For unit tests in this example I use [Alsatian](TBD). Alsatian requires me to first build the Typescript files myself so the examples here should be easily portable to other unit testing frameworks that have a similar requirement. Some unit testing frameworks will do the build for you - I can't help you there.

I will assume that you are reasonably familiar with how to build a single app using Webpack, so I won't go into too much detail about that. You can always refer to the full solution on [GitHub](TBD) to see all the references and packages in use.

## Solution structure
For the purposes of this, I have an example solution (find it on [GitHub](TBD)) like this:
--> Admin.App
--> Admin.Web
--> Customer.App
--> Customer.Web

The `*.App` folders hold the javascript app I am building. The `*.Web` folders would have a web application that can serve up the javascript app as well as provide various APIs etc. This could be ASP.Net Core, Node or whatever. It's not really important, the point is that after I build each app, I want to copy the generated `js` and `css` files to the relevant web folder. There are a million ways you may want to set this up and deploy it in practice, this is just one way.

## Building the Typescript with Webpack
In my example, I have a `src` folder in each App folder and under there I have an `index.tsx`. I want to build the Admin App to ./dist/js/adminapp.js and the equivalent for the Customer App. Note that the `dist` folder lives in the root of the solution. 
You can see the `tsconfig.json` file I use on [GitHub](TBD). The important part is `webpack.config.js` which lives in the root of the solution and looks like this, at this point:
```
const path = require("path");
const webpack = require('webpack');
const { CheckerPlugin } = require("awesome-typescript-loader");

const DIST_DIR = "dist";

const config = {
    entry: {
        adminapp: path.resolve(__dirname, "src/Admin.App/src/index.tsx"),
        customerapp: path.resolve(__dirname, "src/Customer.App/src/index.tsx")
    },
    output: {
        path: path.resolve(__dirname, DIST_DIR),
        filename: 'js/[name].js'
    },
    resolve: {
          extensions: ['.ts', '.tsx', '.js']
    },
    module: {
        rules: [
            {
                test: /\.tsx?$/,
                use: "awesome-typescript-loader",
                include: [path.resolve(__dirname, "src")]
            },
        ]
    },
    plugins: [
        new CheckerPlugin(),
    ]
}

module.exports = config;
```
Most of this is common, the key points are the following:

```
    entry: {
        adminapp: path.resolve(__dirname, "src/Admin.App/src/index.tsx"),
        customerapp: path.resolve(__dirname, "src/Customer.App/src/index.tsx")
    }
```
This tells Webpack that there is more than one app to build seperately; it effectively sets up multiple build processes.

The other important part is 
```
    output: {
        path: path.resolve(__dirname, DIST_DIR),
        filename: 'js/[name].js'
    },
```
which tells Webpack where to put the output. Note `[name]` parameter which tells Webpack to give each output file a different name. You could probably change that to `[name]/app.js` if you prefer to group the files by app.

The full `package.json` is on [GitHub](TBD), but the key bit is just 
```
  "scripts": {
    "build": "webpack"
  },
```
which allows me to `npm run build`, which will then build the js files into the `dist` folder. 

Later on we'll get onto how we can copy the resulting files to the different Web folders.


## Building the SCSS
We are likely to have some shared styles as well as some styles that are specific to each App.
In order to handle that, we are first going to put a Style folder in `/src/Style` at the Solution level and another `Style` folder underneath each app. We are then going to put an `index.scss` in each of the three `Style` folders. The one at the Soluton will hold or import shared styles and the one inside each app will only have styles specific to that solution.

In order for Webpack to pick it up, we need to add the following to each of our `index.tsx` files:
```
import "../../Style/index.scss";
import "../Style/index.scss";
```

In order for Webpack to pick this up, we then need to add the `extract-text-webpack-plugin` NPM package (`npm install -D extract-text-webpack-plugin`)

```
const ExtractTextPlugin = require("extract-text-webpack-plugin");

const extractSass = new ExtractTextPlugin({
    filename: "css/[name].css"
});
```

## Moving the resulting files

## Running linters

## Running the unit tests






# Sharing components between apps

But, here is how to use the Paths from tsconfig in webpack so you don't have to specify them twice: https://github.com/webpack/webpack/issues/4160#issuecomment-314792520

Fucking great.... using TSC to compile the test files compiles fine, but the alias remains in the output file, meaning it can't be run   
https://github.com/Microsoft/TypeScript/issues/16088

https://github.com/mazhlekov/tsc-resolve provides a half-decent solution

The one thing I have not dealt with above is the need to share components between apps; It usually makes sense to have an `Common.App` with some shared components that all the apps can use. Before we combined everything into a single package.json, we could simple use a FILE reference to point to the `Common.App` package in each app's `package.json`. Of course, now that we have a single `package.json`, that no longer works. We can, of course, use full relative paths to the common functions, but that is brittle. Making it work more sensibly is a bit involved, so I have put that in a [separate post](TBD).
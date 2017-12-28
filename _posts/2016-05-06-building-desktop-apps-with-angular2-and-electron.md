---
layout: post
title: "Building Desktop apps with Angular2 and Electron"
description: "Build incredible desktop experiences with electron and angular"
thumb_image: "posts/electron.jpg"
tags: [web, angular, javascript]
---

{% include image.html path="posts/electron.jpg" path-detail="posts/electron.jpg" alt="Electron and Angular" %}

[Electron](http://electron.atom.io/) is a great open source tool from [Github](https://github.com/) that allows you to leverage your Javascript skills to build Desktop apps for MacOS, Windows and Linux systems. If you are using something like Slack, VisualStudio code or Atom you are already using Electron in your environment. Chances are you are running one or two of these apps but they are so smooth you can’t really tell any difference from a pure native desktop app.

Making Desktop apps is generally harder than making web apps, and given the fact that you have to target 3 different OS’s it makes it even harder. But if your app works on the web it will work with electron. Electron has a built in **Chromium** browser that it uses to serve your application and its unique ability to target different OS’s services via the electron API allows you to do things like Native desktop notifications in a very easy way and package your app properly for any OS.

## How electron works

Electron runs Node.js together with Chromium to create a shell for the app, and this allows Electron to do lower level interactions with the OS in a way normal browsers can’t. And because its running chromium all the typical javascript / CSS / HTML you write for a normal web app works perfectly.

## Building an electron app with Angular2

We said that electron works with any Javascript app, and naturally it works nice with angular2 as well. So let’s dive into how we can build an angular2 app with electron and go through the steps necessary to do this.

I will be using webpack and base myself on [this repo](https://github.com/joaogarin/angular2-electron) that I have made that does a couple more things that I am doing here like authenticating with github. You might want to check the code in there. At this time I am using rc0 release.

## Creating a package.json

First things first, we need to import dependencies and setup or package.json so that via [npm](https://www.npmjs.com/) we install everything we need.

The important part of the package.json looks like this : 

{% highlight javascript %}
...
"dependencies": {
    "@angular/http": "2.0.0-rc.0",
    "@angular/common": "2.0.0-rc.0",
    "@angular/compiler": "2.0.0-rc.0",
    "@angular/core": "2.0.0-rc.0",
    "@angular/platform-browser": "2.0.0-rc.0",
    "@angular/platform-browser-dynamic": "2.0.0-rc.0",
    "@angular/platform-server": "2.0.0-rc.0",
    "@angular/router": "2.0.0-rc.0",
    "@angular/router-deprecated": "2.0.0-rc.0",
    "core-js": "^2.2.2",
    "rxjs": "5.0.0-beta.6",
    "zone.js": "0.6.11",
    "electron-prebuilt": "^0.35.4",
    "webpack-target-electron-renderer": "^0.4.0"
  }
...
{% endhighlight %}


In the repository you will find some other things like development dependencies so that ts files can be compiled and redux for managing state. But these are really the important pieces that you have to have in your package.json file.

Other interesting part of the package.json file is the tasks. They look like this : 

{% highlight javascript %}
...
"scripts": {
    "watch": "npm run watch:dev",
    "watch:dev": "webpack --watch --progress --profile --colors --display-error-details --display-cached",
    "build": "npm run build:dev",
    "build:dev": "webpack --progress --profile --colors --display-error-details --display-cached",
    "package": "node package.js",
    "package-all": "npm run package -- --all",
    "typings-install": "typings install",
    "postinstall": "npm run typings-install",
    "electron": "electron src/app"
  }
...
{% endhighlight %}

You can see the full file [here](https://github.com/joaogarin/angular2-electron/blob/master/package.json)

For now let’s focus on the build task and the electron. It just compiles the app with the default webpack.config.js file and the electron task simply runs electron and the source of the electron app, in this case **src/app**.


## Webpack config

The webpack file is pretty straight forward, and if you have been using webpack with angular2 or have looked at [Angular class starter] (https://github.com/AngularClass/angular2-webpack-starter) it probably looks familiar.

The only specific thing here out of any other ordinary webpack config file is that we are targeting electron using the **webpack-target-electron-renderer** package.

{% highlight javascript %}
...
/**
 * Target Electron
 */
config.target = webpackTargetElectronRenderer(config);
module.exports = config;
...
{% endhighlight %}

Other than that we are telling webpack to compile our app into a app.js file that will be used by electron. We also compile some polyfill and vendor packages like rxjs using the CommonsChunkPlugin plugin from webpack.

{% highlight javascript %}
...
// our angular app
    entry: {
        'polyfills': './src/polyfills.ts',
        'vendor': './src/vendor.ts',
        'app': './src/app/app'
    },

    // Config for our build files
    output: {
        path: helpers.root('src/app/dist'),
        filename: '[name].js',
        sourceMapFilename: '[name].map',
        chunkFilename: '[id].chunk.js'
    },
…
{% endhighlight %}

You can see the full file [here](https://github.com/joaogarin/angular2-electron/blob/master/webpack.config.js)

## Building the electron app

Everything for our app will live inside the folder src/app and as we saw that is where the electron script will be looking to find our app. So let’ start creating the app and everything necessary for it.

In this folder we will need a package.json file as well, but this one will be used only by electron, it will provide some basic information like the name, version and where the entry point of the app is. The electron script will look for this file to bootstrap the electron app. This file looks like this : 

{% highlight javascript %}
{
  "name": "app",
  "version": "0.0.1",
  "main": "main.js"
}
{% endhighlight %}

We then need to create our main.js file. This file tells electron where the app is, where the entry html file is and provides some basic methods for initializing and closing our app when the user closes the window. This is the full file :

{% highlight javascript %}
/**
 * Include our app
 */
var app = require('app'); 
// browser-window creates a native window
var BrowserWindow = require('browser-window');
var mainWindow = null;

app.on('window-all-closed', function () {
  if (process.platform != 'darwin') {
    app.quit();
  }
});

app.on('ready', function () {
  // Initialize the window to our specified dimensions
  mainWindow = new BrowserWindow({ width: 1200, height: 900 });
  // Tell Electron where to load the entry point from
  mainWindow.loadURL('file://' + __dirname + '/index.html');
  
  // Open the DevTools.
  mainWindow.webContents.openDevTools();
  // Clear out the main window when the app is closed
  mainWindow.on('closed', function () {
    mainWindow = null;
  });
});
{% endhighlight %}

Next step is to create our angular app. This will be just a normal angular app. Create an app.ts file inside this folder with the following : 

{% highlight javascript %}
/*
 * Providers provided by Angular
 */
import {provide} from '@angular/core';
import {bootstrap} from '@angular/platform-browser-dynamic'
import {LocationStrategy, HashLocationStrategy} from '@angular/common';
// Decorators
import {Component} from '@angular/core';

/*
 * App Component
 * Top Level Component
 */
@Component({
    // The selector is what angular internally uses
    selector: 'app', // <app></app>,
    template: `
    <div>
        Hello Electron
    </div>
    `
})
export class App {}

/*
 * Bootstrap our Angular app with a top level component `App` and inject
 * our Services and Providers into Angular's dependency injection
 */
bootstrap(App, [
    provide(LocationStrategy, { useClass: HashLocationStrategy })
]).catch(err => console.error(err));
{% endhighlight %}

We will also need an index.html file.

{% highlight html %}
<!DOCTYPE html>
<html lang="">

<head>
    <title>
        Angular 2 Electron
    </title>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="description" content="Simple electron angular2">
</head>

<body>
    <app>
        Loading...
    </app>
    <script src="dist/polyfills.js"></script>
    <script src="dist/vendor.js"></script>
    <script src="dist/app.js"></script>
</body>
</html>

{% endhighlight %}

## Running the app

You can now run :

{% highlight bash %}
 npm run build
{% endhighlight %}

And that will build the app. And if after you run 

{% highlight bash %}
 npm run electron
{% endhighlight %}

that will start electron with your app.

You now have an angular2 app running inside electron. You will notice that is basically looks pretty much like a Chrome browser running on a destkop window. And that is more or less what is happening.

You can now head over to [this repo](https://github.com/joaogarin/angular2-electron)
 Where I am doing a little more than just this. Here I am doing a basic connection to the github api.

You need to provide a config.json file inside the **src/app** folder that should look like : 

{% highlight javascript %}
{
    "github": {
        "client_id": "yourclientID",
        "client_secret": "yoursecretkey",
        "scopes": [
            "user:email",
            "notifications"
        ]
    }
}
{% endhighlight %}

Notice the warning on the repo : 

***Don’t use this in production as for production you should have a safe server side URI and not have your secret key in the app folder.***

However this is a good way to start playing around with the browserWindow object from electron and other things that electron provides. You can check more on this in the [electron documentation](http://electron.atom.io/docs/v0.37.8/)

## Packaging

Packaging your apps in electron is also an easy thing to do. There are several libraries that help you do this in an automated way via a simple js script task. One of those is [electron-packager](https://github.com/electron-userland/electron-packager)

You can configure all sorts of things, target different devices and do all sorts of things, you can find the documentation on their github page.

## Wrapping up

Electron is a very useful tool, I must admit that desktop apps has been something kind of unknown for me for a long time, although I enjoy some of these apps like slack I always thought it would be just too much pain to build on of those. Electron opens a new world of possibilities and it’s incredibly easy and fun to build with.

I encourage you to have a look and try out a sample app and you will see how easy it is to start building with it as soon as you pass the initial boilerplate associated with it which I described here in this blog post.

Some useful links : 

*   [Auth0 image uploader app](https://auth0.com/blog/2015/12/15/create-a-desktop-app-with-angular-2-and-electron/)
*   [My github auth repo](https://github.com/joaogarin/angular2-electron)
*   [Electron documentation](http://electron.atom.io/docs/v0.37.8/)

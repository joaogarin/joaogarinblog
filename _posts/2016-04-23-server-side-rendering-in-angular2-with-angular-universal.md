---
layout: post
title: "Server side rendering in Angular2 with Angular Universal"
description: "Server rendering Angular applications using Universal"
thumb_image: "posts/server_0.jpg"
tags: [web, angular, javascript]
---

{% include image.html path="posts/server_0.jpg" path-detail="posts/server_0.jpg" alt="Angularjs components" %}

Angular 2 comes with a lot of new and awesome things. One of which is the separation between the *application layer* and the *render layer*. This leads to the possibility to do a lot of great things, one of which is Server Side Rendering.


## Angular Universal

For the purpose of getting server side rendering in Angular2 [Jeff Whelpley](https://twitter.com/jeffwhelpley) and [PatrickJs](https://twitter.com/gdi2290) created the [Angular Universal project](https://github.com/angular/universal), this project is separated from Angular2 core itself but makes it super easy to plugin into your existing Angular2 application and start getting server side rendering to work for you. The specifications and basic design for the project can be found in the [Google docs spec](https://docs.google.com/document/d/1q6g9UlmEZDXgrkY88AJZ6MUrUxcnwhBGS0EXbVlYicY/edit)


## Why Server Side rendering

There might be different reasons for why you would want **Server Side Rendering** in your web application and I will not get too much into why it is a good thing but I will list some of the important things that were the fundamental reason for the existence of such a project, and also why other frameworks like **Meteor** or **React** provide server side rendering for web apps :

*   Performance (Perceived load time and actual load time)
*   SEO 
*   Browser Support
*   Load preview

You can read a little more into each of these topics in the google doc in [this section](https://docs.google.com/document/d/1q6g9UlmEZDXgrkY88AJZ6MUrUxcnwhBGS0EXbVlYicY/edit#heading=h.je8lg3vxthdp)


## How it works

The solution designed for Angular Universal is actually quite interesting and differs a little from what other frameworks do to provide server side rendering.
The basis of the server side rendering architecture in Angular2 includes two essential steps : 

*   Rendering on the server
*   Transitioning server rendered into client side

For the first one it was quite obvious to take advantage of the rendering layer of Angular2 since it is already completely separated from the *DOM*.

The second step was where a really nice project got to come to life called **Preboot**


### Preboot

Preboot essentially records events on the server side view and then plays them back once the the client app kicks in and becomes available.  You can read more about it [here](https://docs.google.com/document/d/1q6g9UlmEZDXgrkY88AJZ6MUrUxcnwhBGS0EXbVlYicY/edit#heading=h.b3qdlffhbqmu) and I definitely recommend you do because it's a very interesting solution to a very hard problem. Preboot is highly configurable and was created outside of the scope of Angular2 so it can actually be used by any framework or application.

Key features of preboot include the following.

*   **Record and playback events** - You can configure through preboot options which actions / events are to be recorded and how they should be replayed
*   **Respond immediately to events** - For situations where you want something to be immediately available and not wait for the client to become “live” before the feature is responsive.
*   **Maintain focus even page is re-rendered** - Keeping the focus on a specific element (like a textbox) when the client view becomes available, event if a page re-render is necessary.
*   **Buffer client-side re-rendering for smoother transition** - The might be cases where you want to buffer how the client side view becomes available for a smoother transitioning from server side view to client side view.
*   **Freeze page until bootstrap complete if user clicks button** - if enabled as an option when a button is clicked the page freezes until client side becomes fully available and processes that button event.

A lot of other frameworks like **React** don’t have such a system where your server side view is already interactive for client side events and in these cases this connection between the server side rendered view and client side simply does not exist.

### Workflow 

This is how the workflow for Angular Universal looks like : 

1. HTTP GET request sent to the server
2. Server renders page that contains the following: Rendered HTML from ServerDomRenderer that the user will see initially and Inline JavaScript for preboot (see ‘preboot’ section below)
3. Browser receives initial payload from server
4. User sees server view
5. Preboot starts recording events 
6. Requests made for additional external images, JS, CSS, etc.
7. Once external resources loaded, Angular client bootstrapping begins
8. Client view rendered
9. Bootstrap complete, so Angular client calls preboot.done()
10. Preboot events replayed in order to adjust the application state to reflect changes made by the user before Angular bootstrapped (i.e. typing in textbox, clicking button, etc.)


## The code 

Like mentioned previously it is really simple to port a client side only app to a server side app with Angular Universal. And in code it looks something like the following (I will show only the important steps here, a good solution and example can be found in the [Universal starter repository on github](https://github.com/angular/universal-starter)

### It always starts with an index.html

{% highlight html %}
<!doctype html>
<html lang="en">
<head>
  <title>Angular 2 Universal</title>
  <meta charset="UTF-8">
  <meta name="description" content="Angular 2 Universal">
  <meta name="keywords" content="Angular 2,Universal">
  <link rel="icon" href="data:;base64,iVBORw0KGgo=">
  <base href="/">
</head>
<body>
  <app>
    Loading...
  </app>
  <script src="/dist/client/bundle.js"></script>
</body>
</html>
{% endhighlight %}

This client bundle is really just a compilation of our client bootstrap that I will show bellow in the client.ts file and the app component which for this is really not important. It can be any app, for now just think of it as your own Angular2 app.

### Server Bootstrap

 First important thing is that the Universal project provides its own *bootstrap* and *bindings* so you will need to include them in your *server.ts* file.

{% highlight javascript %}
import 'angular2-universal/polyfills';
import {
  provide,
  enableProdMode,
  expressEngine,
  REQUEST_URL,
  ORIGIN_URL,
  BASE_URL,
  NODE_ROUTER_PROVIDERS,
  NODE_HTTP_PROVIDERS,
  ExpressEngineConfig
} from 'angular2-universal';
{% endhighlight %}

You can then start a server engine such as express to deliver your app server side. This looks just like any normal *express app* 

{% highlight javascript %}
const app = express();
const ROOT = path.join(path.resolve(__dirname, '..'));
// Express View
app.engine('.html', expressEngine);
app.set('views', __dirname);
app.set('view engine', 'html');
{% endhighlight %}

In the third step you then provide the express app with a configuration for the universal app including directives, providers such as HTTP or other providers your app might require.

{% highlight javascript %}
function ngApp(req, res) {
  let baseUrl = '/';
  let url = req.originalUrl || '/';
  let config: ExpressEngineConfig = {
    directives: [ Html ],
    platformProviders: [
      provide(ORIGIN_URL, {useValue: 'http://localhost:3000'}),
      provide(BASE_URL, {useValue: baseUrl}),
    ],
    providers: [
      provide(REQUEST_URL, {useValue: url}),
      NODE_ROUTER_PROVIDERS,
      NODE_HTTP_PROVIDERS,
    ],
    async: true,
    preboot: false // { appRoot: 'app' } // your top level app component selector
  };
  res.render('index', config);
}
{% endhighlight %}

We also tell express to render our app from the index.html above

{% highlight javascript %}
// Serve static files
app.use(express.static(ROOT, {index: false}));
{% endhighlight %}

You can also pass in routes to the app by doing the following 

{% highlight javascript %}
// Routes
app.use('/', ngApp);
app.use('/about', ngApp);
app.use('/home', ngApp);
{% endhighlight %}

Finally you make your app listen on a specific *port*. This will launch your express app and put it live under this port.

{% highlight javascript %}
// Server
app.listen(3000, () => {
  console.log('Listen on http://localhost:3000');
});
{% endhighlight %}

This might be a good time and look at the starter full [server.ts file](https://github.com/angular/universal-starter/blob/master/src/server.ts)


### Client bootsrap and preboot done()

On our *client.ts* file the code looks maybe more familiar to a normal Angular2 app with one detail that we mentioned earlier. The client needs to tell **preboot** that it finished bootstrapping and that preboot can then start playing back the events that recorded while the client was *bootstraping*, also bindings are imported from angular-universal like in the server example.

This is actually the whole *client.ts* file : 

{% highlight javascript %}
import 'angular2-universal/polyfills';
import {bootstrap, enableProdMode, BROWSER_ROUTER_PROVIDERS, BROWSER_HTTP_PROVIDERS} from 'angular2-universal';
import {App} from './app/app.component';
enableProdMode();
bootstrap(App, [
  ...BROWSER_ROUTER_PROVIDERS,
  ...BROWSER_HTTP_PROVIDERS
]);
{% endhighlight %}


## The app 

In all of this your question about this might be “What about the app? Should anything Universal specific be done there?”. The answer to this is “no” but there are some things that is important to be aware of.

The app remains just an Angular2 app and nothing Universal specific is done there. But there are some best practices for angular universal that should be taken in consideration when building the app.

There are especially two key aspects of a server side app to take in mind.

### Don’t manipulate native elements directly. Use the rendered element instead

This looks something like this 

{% highlight javascript %}
constructor(element: ElementRef, renderer: Renderer) {
  renderer.setElementStyle(element.nativeElement, 'font-size', 'x-large');
}
{% endhighlight %}

### Know the difference of attributes and properties and how they relate to the DOM

It’s important that you know the difference between attributes and properties once the browser parses your HTML. [A good question on stackoverflow regarding this](http://stackoverflow.com/questions/6003819/properties-and-attributes-in-html)

It’s especially important maybe to know that relation between attributes and properties and attributes is not always a one-to-one relationship as shown in the value attribute example.

### Don't use any of the browser types provided in the global namespace

Global namespace browser types like document or navigator don’t exist outside of the browser so when trying to get access to them when doing server side rendering in nodejs will result in some errors.


### Keep your directives stateless as much as possible

For stateful directives, you may need to provide an attribute that reflects the corresponding property with an initial string value such as url in img tag. For the native ```<img src="">``` element the src attribute is reflected as the src property of the element type HTMLImageElement.


## Wrapping up

There are many exciting things happening with Angular2, server side rendering is definitely one of them. 

At this moment Angular Universal is at version *0.98* so there are still quite a lot of small issues open, but it already provides a really stable solution for server side rendering. If you are reading this than it looks like it works as this website is running on express with angular universal fetching data from a CMS in the background. 

Its already much better than anything we had before that’s for sure. You can follow the issues in the repo [here](https://github.com/angular/universal/issues)

But the options and flexibility that come with the render layer of Angular2 does not stop at server rendering. You might also be interested and read about **Service workers**, **Web Workers** or **Angular2 with Electron** to deliver desktop apps, or even **Angular2 with NativeScript** for mobile native app development. Bellow are some links to some of these topics :

*   [Web Workers in angular2](https://github.com/angular/angular/blob/master/modules/angular2/docs/web_workers/web_workers.md)
*   [Progressive angular app](https://github.com/angular/progressive)
*   [Create a desktop app with angular2 and electron](https://auth0.com/blog/2015/12/15/create-a-desktop-app-with-angular-2-and-electron/)
*   [NativeScript and angular2](https://nativescript.github.io/nativescript-angular-guide/)
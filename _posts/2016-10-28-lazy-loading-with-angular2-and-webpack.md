---
layout: post
title: "Lazy loading with Angular2 and Webpack"
description: "Let’s analyze how NgModule() works, what it is and how it can help to enable lazy loading in Angular 2 applications today. In this case we will be using Webpack"
thumb_image: "posts/routes.jpg"
tags: [web, angular, javascript]
---

{% include image.html path="posts/routes.jpg" path-detail="posts/routes.jpg" alt="routes" %}

Right before the final release of Angular 2, [NgModule()](https://angular.io/docs/ts/latest/guide/ngmodule.html) was introduced. This was kind of a surprise for some people (myself included) but there was a higher purpose to the inclusion of such a concept, and that was to enable easy lazy loading of these modules.


Let’s analyze how NgModule() works, what it is and how it can help to enable lazy loading in Angular 2 applications today. In this case we will be using [Webpack](https://webpack.github.io/).


## NgModule()


NgModule() allows us to *wrapp* together components, directives, providers, services and other modules and group them together, creating sort of a block of a cohesive block of functionality. It’s sort of like it was in Angular 1, and if you are now coming to Angular 1 perhaps this will be obvious but for a lot of the Alpha and even Beta and RC period of Angular 2 such a concept did not exist, and going back to it felt a little strange for most part.


Technically it’s a class decorated with **@NgModule**, that takes some metadata object that tells Angular how it should compile and run that code.


I am not going to go into what kind of modules there are and you should have in your app, the official [Angular 2 docs](https://angular.io/docs/ts/latest/guide/ngmodule.html) does a great job of explaining that. I am going to focus on how to do lazy loading with NgModule().


One important aspect of Modules is that they can define their own routes, and using this system we can easily setup modules to be lazy loaded when a URL of a particular module is visited.


## Overview of the app


So we have an app, it has 2 modules let’s assume. It has a main **AppModule** where we bootstrap our app, and it has a **ProjectsModule** that we want to lazy loading when someone goes for a route that’s on that module’s route definitions. We also have a **SharedModule** that contains some common utility components and services that we want to use on both main **AppModule** and **ProjectsModule**


## Setting up main NgModule() with routes


This is the definition of our main app NgModule()


{% highlight javascript %}
...
export const routes: Routes = [
  { path: '', component: HomeComponent },
  { path: 'home', component: HomeComponent },
  {
    path: 'projects', loadChildren: () => System.import('./components/container/projects').then((comp: any) => {
      return comp.default;
    })
    ,
  },
];


@NgModule({
    imports: [
        BrowserModule,
        SharedModule.forRoot(),
        RouterModule.forRoot(routes, { useHash: true }),
    ], declarations: [
        AppComponent,
        HomeComponent,
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
...
{% endhighlight %}


It imports the BrowserModule and the SharedModule (ignore for now the forRoot() call on the sharedModule we will go over this in the end of the article), it also declares some components and bootstraps on the AppComponent. But let’s focus on the declaration of the routes for a moment.


This main NgModule() defines some routes, and a specific route (the one we want to lazy load) we use the following route definition : 


{% highlight javascript %}
...
{
    path: 'projects', loadChildren: () => System.import('./components/container/projects').then((comp: any) => {
      return comp.default;
    })
    ,
  },
...
{% endhighlight %}


By using **loadChildren** we are telling Angular that we want to lazy load a module into this route (/projects) and that module will define the *children* of this route, meaning anything after “/projects” will be defined by the module inside the path **'./components/container/projects'**.


And what’s the **System.import()** part in here doing? That’s the magic telling Webpack 2 to create and load this bundle when this route is navigated to. This module needs to be exported using a [default export](https://developer.mozilla.org/en/docs/web/javascript/reference/statements/export#Using_the_default_export) inside this file for webpack to understand and load it properly using System.import


## Setting up the projects module


Now that we have things ready in our main AppModule it’s time to define our lazy loaded module, the Projects module. This module will define all routes that will appear under */projects* and will import its own components, directives etc..


This is how the definition for this module looks like : 


{% highlight javascript %}
...
export const routes: Routes = [
    { path: '', component: NewProjectComponent, pathMatch: 'full' },
    { path: 'new-project', component: NewProjectComponent },
    { path: 'detail/:id', component: ViewProjectComponent },
];


@NgModule({
    imports: [
        CommonModule,
        SharedModule,
        RouterModule.forChild(routes),
    ],
    declarations: [
        // Components / Directives/ Pipes
        EditProjectComponent,
        NewProjectComponent,
        ViewProjectComponent,
    ],
})
export default class ProjectModule {
    static routes = routes;
}
...
{% endhighlight %}


Similarly to the main module the **ProjectsModule** imports the Modules it needs (the **CommonModule** and the **SharedModule**) and declares some components, in this case three components.


It also defines it’s own Routes, one thing to notice is that each of these routes is accessible through */projects/the-route-subpath* as in the main module we said that we would be *“loading children”* for the projects module, hence the usage of **RouterModule.forChild(routes)** when defining the lazy loaded module. 

So the actual URL for these routes are : 


*   **/projects** - the default route for the projects module
*   **/projects/new-project** - The second route we defined in the projects module
*   **/projects/detail/:id** - The third route we defined, this one takes a parameter which is the project id




Notice one important thing we mentioned earlier, due to how System.import() works we need to export the Module using default exports for the modules we want to lazy load, in this last snippet we see : 


{% highlight javascript %}
export default class ProjectModule
{% endhighlight %}


If you don’t export it using default export webpack will not be able to understand and load your module at runtime using System import.


And that’s it. There is nothing else you would need to do to have the ProjectsModule loaded lazily when users visit the app. 




## Summary


Let’s briefly analyze the points that one needs to take in order to have lazy loading working with NgModule()


1 ) Split up the app and the lazy modules using NgModule()
2 ) Setup routes for the main module using RouterModule.forRoot(routes), and *loadChildren*for the lazy parts
3 ) use System.import() to load the children 
4 ) Setup the lazy module using RouterModule.forChild(routes),
5 ) Export the lazy module using default exports


## SharedModules


One common problem people get into when scaling an app is that there are components, directives and more importantly services that need to be shared across an app in different ways and for that **SharedModules** come in handy. 


Being able to put these things in a single shared module that can be used across the app is extremely useful but has a couple of *gotchas* that are quite important and can lead to  some unexpected behaviour, especially when it comes to lazy loaded modules and the Dependency injection tree that Angular creates.


### Importing and exporting modules in a SharedModule


The first thing that is important to understand is that a **SharedModule** can import modules that it needs for it’s components, a common case of this is the **CommonModule** that exports directives like *ngFor* and *ngClass*. But more importantly it can export modules and the module that a **SharedModule** exports will be available for any module that imports this **SharedModule**.


A **SharedModule** can even export a module without importing it, for example a **SharedModule** can export **FormsModule**, even though it does not import it solely for the purpose of making that **FormsModule** available for any module that imports it. You can find information about this behaviour in the [SharedModule documentation](https://angular.io/docs/ts/latest/guide/ngmodule.html#!#shared-module)


## Sharing Services / Providers in a SharedModule


When it comes to providers the behaviour is a bit more complex. When we include a provider in a **SharedModule** and we import that **SharedModule** a duplicate instance of that provider will be created. 


This is an intended behaviour of the Angular 2 Dependency Injection tree, see [Dependency Injection docs](https://angular.io/docs/ts/latest/guide/dependency-injection.html) but that can lead to some unexpected results creating duplicate instances of services in an Angular app when a modue is lazy loaded and imports a service included in a SharedModule.


When we want a *Singleton* service (application-wide service) to be available everywhere in the app it should be included in the app main Module (AppModule in the previous example) where we bootstrap the app. This insures that the service will be available in all modules of the app including lazy loaded modules.


However, if we really want to we can still included these services in a **SharedModule** using again the *forRoot()* behaviour similar to what we did with the router.


## Defining forRoot() in a SharedModule


So to make sure that a provider included in the **SharedModule** is not duplicated in a lazy loaded module that imports the **SharedModule** we need to do a couple of things


### 1 ) Include providers in the forRoot() method 


The first thing we need to do is to register the providers in the forRoot() method inside the **SharedModule** class instead of registering it in the **NgModule()** object *providers* property where we normally register providers.


This looks like the following : 


{% highlight javascript %}
...
@NgModule({
    imports: [
        CommonModule,
    ],
    declarations: [
        ButtonComponent,
    ],
    exports: [
        ButtonComponent
    ],
})
export class SharedModule {
    static forRoot(): ModuleWithProviders {
        return {
            ngModule: SharedModule,
            providers: [
                APIService,
            ]
        }
    }
}
...
{% endhighlight %}


Here is a sample **SharedModule** that imports CommonModule and declares and exports a generic ButtonComponent. It registers the *APIService* service inside the forRoot() method and lets look at how each Module includes this **SharedModule** so that a duplicate instance of APIService is not created.


### 2 ) Include SharedModule.forRoot() inside AppModule


In our main module where we want this service to be available and where we want the instanace of **APIService** to be created as a singleton (application-wide service) we use the forRoot() when including **SharedModule**. We saw this before when we defined our **AppModule**.


{% highlight javascript %}
...
@NgModule({
    imports: [
        BrowserModule,
        SharedModule.forRoot(),
        RouterModule.forRoot(routes, { useHash: true }),
    ], declarations: [
        AppComponent,
        HomeComponent,
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
...
{% endhighlight %}


This makes sure that this service is included in the instance of the **SharedModule** we are creating. This is like saying **“give me everything in the **SharedModule** including whatever is defined inside the forRoot() method”**


### Normally include SharedModule inside the lazy loaded module


In our lazy loaded module nothing changes, we just include the **SharedModule** normally, but because the provider was registered in the forRoot() method it will not be included, and so a duplicate instance of it will not be created.


This looks just like previously when we included the **SharedModule** in our **ProjectsModule**


{% highlight javascript %}
...
@NgModule({
    imports: [
        CommonModule,
        SharedModule,
        RouterModule.forChild(routes),
    ],
    declarations: [
        // Components / Directives/ Pipes
        EditProjectComponent,
        NewProjectComponent,
        ViewProjectComponent,
    ],
})
export default class ProjectModule {
    static routes = routes;
}
...
{% endhighlight %}


Here the **SharedModule** will be included but whatever is registered inside forRoot() will not be included.


[This video](https://www.youtube.com/watch?v=SBSnsNHQYo4) from Angular University explains this behaviour in great detail.


## Wrapping up

That’s it. We have lazy loading working with our Projects feature module, it’s a bit to take in at the beginning but taking out the part of services and the Dependency Injection behaviour the process is as easy as lazy loading can get.


A big thanks to the [AngularClass](https://angularclass.com/) and [PatrickJs](https://twitter.com/gdi2290) by providing good insight in this in [their starterkit](https://github.com/AngularClass/angular2-webpack-starter)


Here are some useful links on the subject : 


[Sharing services in lazy modules](https://www.youtube.com/watch?v=SBSnsNHQYo4)
[NgModule docs](https://angular.io/docs/ts/latest/guide/ngmodule.html)
[Angular2 Dependency Injection](https://angular.io/docs/ts/latest/guide/dependency-injection.html)
[Angular2 webpack starterkit by AngularClass](https://github.com/AngularClass/angular2-webpack-starter)

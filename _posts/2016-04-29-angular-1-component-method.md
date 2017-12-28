---
layout: post
title: "Angularjs Component method"
description: "Analyzing how to use components with angular 1.5+"
thumb_image: "posts/comp.jpg"
tags: [web, angular, javascript]
---

{% include image.html path="posts/comp.jpg" path-detail="posts/comp.jpg" alt="Angularjs components" %}

If you are anything like me, you love markup and HTML. It was one of the first things that got me into web development in the first place, the idea that with some of this syntax I could bring ideas to life in the browser and put it together with CSS and Javascript and create real experiences for users in apps or websites. 

Now if you are really like me you like one specific type of markup and that is *semantic markup*. Markup that really makes sense and tells the story when you see it instead of distracting you will the layout implementation details. Think of it this way. What looks best to you :

## Normal markup

{% highlight html %}
<div class=”friend”>
  <div class=”friend__image”>
  </div>
  <div class=”friend__name”>
  </div>
</div>
{% endhighlight %}

## Semantic markup

{% highlight html %}
<friend name=”friend-name” image=”friend-image”></friend>
{% endhighlight %}

Call me crazy I do really like the second one so much better. This is a very simple example but think of more complex set of inner markup for this “friend” item and it takes this into a much more drastic proportion.

## Enter Angular

This was when Angular striked to me as a game changer. I could actually start doing this by creating directives in Angular1 and start using semantic markup in my code, and it just made the code look so good and so much simpler than what it normally would.

This is what a typical directive would look like

{% highlight javascript %}
angular.module('docsSimpleDirective', [])
.controller('Controller', ['$scope', function($scope) {
  $scope.customer = {
    name: 'Naomi',
    address: '1600 Amphitheatre'
  };
}])
.directive('myCustomer', function() {
  return {
    template: 'Name: {{customer.name}} Address: {{customer.address}}'
  };
});
{% endhighlight %}

You can then do this in your HTML 

{% highlight html %}
<div ng-controller="Controller">
  <my-customer></my-customer>
</div>
{% endhighlight %}

That looks quite good right? That’s what I am talking about. Using markup that makes sense and really explains what's under it. without giving all the layout implementation details.

**Note :** I am taking this snippet from the [documentation of angular1 for directives](https://docs.angularjs.org/guide/directive) so it has a couple of things you should not be doing like reaching to $scope but that is out of the purpose of this blog post.

### But directives are hard

While that above sample looks pretty straightforward and okay to understand. It already introduces the concept of controller which is where the data actually comes from and all of the sudden you have 3 things you need to make in order to have this little semantic markup. Sounds a little bit too much.

But when I say that directives are hard it’s not just that you have to actually make this but they are really kind of hard to understand and some of what I think makes them complex are : 

*   Creating new scope (Isolated scope) vs inheriting scope from the controller
*   Bind to element, attribute, class name or comment with the **restrict** key word
*   Link, PreLink, PostLink functions can make it confusing

## Angular 1.5 and the component() method

Recently many frameworks have adopted the component approach, where you can compose your app of these simple components that contain other components. It makes apps easier to reason about, simpler to write, more fun and intuitive. It’s all wins.

Angular 1.5 introduces components with the component() method. Put this together with one way data binding also already introduced into Angula 1.5 as well and this pretty much puts Angular 1 back on track with other frameworks, even Angular 2 you might say. 

Simple dumb components that are composable, get data from some kind of input and render layout, eventually emitting some events to the outside world.

And this is how an app based on components looks like in Angular 1.5

### Index.html

{% highlight html %}
<!-- components match only elements -->
<div ng-controller="mainCtrl as ctrl">
  <b>Hero</b><br>
  <hero-detail hero="ctrl.hero"></hero-detail>
</div>
{% endhighlight %}

### Main.js

{% highlight javascript %}
angular.module('heroApp', []).controller('mainCtrl', function() {
  this.hero = {
    name: 'Spawn'
  };
});
{% endhighlight %}

### HeroDetail.js

{% highlight javascript %}
function HeroDetailController() {
}
angular.module('heroApp').component('heroDetail', {
  templateUrl: 'heroDetail.html',
  controller: HeroDetailController,
  bindings: {
    hero: '='
  }
});
{% endhighlight %}

### heroDetail.html

{% highlight html %}
<span>Name: {{$ctrl.hero.name}}</span>
{% endhighlight %}

There are a couple of interesting things here and some interesting things under the hood that are worth going over. Let’s take a look at them one by one starting with the index.html.

### Elements only

Notice the comment *“components match only elements”*. That’s exactly what it means, you can not use components as you use directives matching for example attributes or class names. Components bind only to the element itself and that is the only way you should use them, directives on the other hand are meant to decorate and so they can be used in other ways. We will get to some sort of comparison for the two of them in the end.

### module.component(name, options);

Notice the arguments on the component. In directives we have : 

{% highlight javascript %}
module.directive(name, fn);
{% endhighlight %}

Where in the first argument we have the name of the directive and the second argument we have the directive definition function which then returned the directive definition object.

And in components we have 

{% highlight javascript %}
module.component(name, options);
{% endhighlight %}

Where we have the name in the first argument as in directives but in the second argument we have the object definition itself directly in there. This makes it a bit simpler and cleaner when writing components.

### Bindings

In directives we have a *scope* where like we mentioned we can have inherited scope or isolated (new) scope for the directive. Usually and this has kind of become the standard you create an isolated scope for the directives and use *bindToController* to pass in the necessary properties we want to pass from the controller to the directive.

In components we have the bindings keyword which lets us just pass in directly the properties we want from the controller removing the unnecessary boilerplate from having to go through all that scope and bindToController and just assume by default we use an isolated scope.

{% highlight javascript %}
bindings: {
    hero: '='
}
{% endhighlight %}


### controller and controllerAs

Controller is defined in the same way in both worlds but in the component method controllerAs is the default with the keyword $ctrl. 

Now this can depend a little on how and where you get your controller from. Essentially there are the following options :

1 . create the controller on the fly as we have seen before 

{% highlight javascript %}
{
  ...
  controller: function () {}
  ...
}
{% endhighlight %}

2 . Using a component from somewhere else

{% highlight javascript %}
{
  ...
  controller: 'MyCtrl'
  ...
}
{% endhighlight %}

In this case if we want to use controllerAs we can do it by using 

{% highlight javascript %}
...
  controller: 'MyCtrl',
  controllerAs: 'thecontroller'
  ...
{% endhighlight %}

This would allow us to use thecontroller.property inside the template for this component to use certain properties of the controller.

In Angular 1.5 however we can also define controllerAs in the component method in an easier way by doing 

{% highlight javascript %}
{
  ...
  controller: 'SomeCtrl as something'
  ...
}
{% endhighlight %}

Saving us the trouble of having to define controllerAs. But you can still use it in case you want and if you want to keep some backward compatibility with previous versions.

3. Don’t even define any controller

In Angular 1.5 as we said in component() method a controllerAs is set by default to the keyword $ctrl so you can skip defining controller name at all and just do 

{% highlight javascript %}
{
  ...
  controller: function () {}
  ...
}
{% endhighlight %}

and in the template use $ctrl.property to access any property of the controller.


## Other features of component

The component method has a couple more things that are worth going over and that quite honestly I am still discovering while refactoring some apps from directive approach to component approach but let’s go over some of them.

### Require

Just as directives, components can use the require keyword to “require the controllers of other directives to enable communication between each other.” You can read a very interesting article from [Todd Motto](https://twitter.com/toddmotto) on [Directive to Directive communication with require](https://toddmotto.com/directive-to-directive-communication-with-require/)

This will look something like 

{% highlight javascript %}
.component('myComponent', {
  ...
  require: {
    tabsCtrl: '^myTabs'
  },
  ...
});
{% endhighlight %}

### One way Bindings

Like we said earlier to get the full performance and useful restraints in your app development you should start using one way bindings, they were introduced also in Angular 1.5 and you can use them as following 

{% highlight javascript %}
.component('myComponent', {
  ...
  bindings: {
    oneWayBind: '<'
    twoWayBind: '='
  },
  ...
});
{% endhighlight %}


## Are components replacing directives? When to use or not use them?

Components were not introduced to replace directives altogether, but rather to replace on particular use case of directives and make that use case easier to make.
So when do you know when to use component and when to use directives?

Components are like mentioned only one use case of directives and they simplify the creating method of this use case. If you are looking for a clean way to lay your app in a semantic component based way, components are the way to go. They : 

*   Are simple to configure
*   promote best practices like controllerAs and isolated scope
*   Optimized for component based architecture
*   Easier to upgrade to Angular2

when you should **NOT** use components as read in the [angular component documentation](https://docs.angularjs.org/guide/component)

*   for directives that rely on DOM manipulation, adding event listeners etc, because the compile and link functions are unavailable
*   when you need advanced directive definition options like priority, terminal, multi-element
*   when you want a directive that is triggered by an attribute or CSS class, rather than an element

## Wrap up

Components are a really exciting feature of Angular 1.5 as they bring the best practices to defaults and introduce the component based architecture already used in Angular2 into Angular1. This makes it easier to upgrade in the future and put together with one way data binding provide the sanity and right defaults to your apps. 

Give it a try and you will realise how easy it is to get started with them.

I used the following articles as reference for this blog post : 


* [https://docs.angularjs.org/guide/component](https://docs.angularjs.org/guide/component)

* [https://toddmotto.com/exploring-the-angular-1-5-component-method/](https://toddmotto.com/exploring-the-angular-1-5-component-method/)
* [https://toddmotto.com/one-way-data-binding-in-angular-1-5/](https://toddmotto.com/one-way-data-binding-in-angular-1-5/)
* [https://gist.github.com/toddmotto/5b4de6c777d3e446e6410fdadb824522](https://gist.github.com/toddmotto/5b4de6c777d3e446e6410fdadb824522)

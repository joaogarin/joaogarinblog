---
layout: post
title: "NgConf 2016 presents unlikely friends Drupal and angular2"
description: "Most of the people watching ngConf 2016 a couple of weeks ago would not expect to see a PHP CMS logo along side with Angular’s logo in the slides of the Keynote"
thumb_image: "posts/friends.png"
tags: [web, angular, javascript]
---

{% include image.html path="posts/friends.png" path-detail="posts/friends.png" alt="Drupal and Angular" %}

Most of the people watching [ngConf 2016](https://www.ng-conf.org/#/) a couple of weeks ago would not expect to see a PHP CMS logo along side with Angular’s logo in the slides of a [Keynote](https://www.youtube.com/watch?v=gdlpE9vPQFs) at the conference. But that was exactly what happened, when the [weather.com](https://weather.com) channel people went on stage to talk about how they have been using Angular1 for a long time on the most traffic heavy Drupal site on the world, and how they recently adopted Angular2 as well.

But the cooperation between the Drupal community and the Angular community has been established long ago and recently we see it come in many shapes and forms in many projects and I will post links to some of them in the end of the article. From discussions on bringing Angular2 to Drupal core, Angular changing its license to make sure it’s compatible with Drupal’s license, to having Angular2 components rendered in Twig etc etc..

A lot of effort has been made in making them go together well, and this shows a lot of some of the principles of Angular in making sure the platform mixes together and joins forces with other fields for the benefit of the web, whether it’s other Javascript frameworks like Ember and React, or completely different areas of the spectrum like PHP, and in this case Drupal.

## Drupal

Drupal is a PHP based CMS software, completely open source with one of the most vibrant communities you can think of. It has thousands of crontributers, and is now on version 8.1 after some [major strategic changes in terms of the software](https://www.drupal.org/node/1674208) and how its built.

It’s not that common to see PHP CMS’s adventuring on the lands of Javascript frameworks but Drupal has a long history on that after early on embracing Jquery, after that Backbone and possibly now [adopting another one](http://buytaert.net/should-we-decouple-drupal-with-a-client-side-framework)

But enough about Drupal let’s discuss what was presented at the conference and what has been done by the people of [weather.com](https://weather.com) and [MediaCurrent](http://www.mediacurrent.com/).

## Weather.com

Like I said and like it was mentioned in the keynote, [weather.com](https://weather.com) is a very large site with about 100 million page views a month and it was an early adopter of Angular1. They are actually still using Angular1 on the site and for a new project called the Weather Underground they went for Angular2 and did some really nice things to provide a seamless integration between Drupal and Angular2 taking this cooperation to the next level.


## Drupal decouple blocks

[Decoupled blocks](https://www.drupal.org/sandbox/mrjmd/2664138) is a Drupal contributed module that came out of this experience in building Weather Underground with Angular2 and it allows you to easily write Angular2 or React (at the moment) components and use them in your Drupal website.

What happens is more or less that you can build Angular2 components (with nested components and services and whatever they might need) and the output is a Drupal block that you can then put on a specific part of a page or use [panels](https://www.drupal.org/project/panels) to have content administrators change the block position within a page etc. It also does a very nice job in managing multiple component and managing when the app actually needs to be bootstrapped or not, but you can read about it in more detail in the [project page of the module](https://www.drupal.org/sandbox/mrjmd/2664138).

But it’s important to notice that the Angular2 code you write is completely Angular2 pure code and completely independent of Drupal, so you can actually write something without any knowledge whatsoever of how Drupal works. 

[Stephen Fluin](https://developers.google.com/experts/people/stephen-fluin) a Google Developer Expert actually went last weel to Drupal Con New Orleans and demoed this approach and the video can be found [here](https://www.youtube.com/watch?v=PGneK_G5zhA&index=20&list=PLpeDXSh4nHjTY67K6N8RZHEiJhQL-dnUf)

## Fully decoupled vs Progressively decoupled

This is what is considered to be a [progressively decoupled approach](http://buytaert.net/the-future-of-decoupled-drupal) where you take small parts of the Drupal site and decoupled them from Drupal but still leverage on most of the Drupal framework things like live preview, big pipe and so on.. 

It is the oposite of a “Fully decoupled” approach where people can use Drupal just as an API using for example [GraphQL](http://graphql.org/) (drupal module comming soon by the way) or [REST](https://www.drupal.org/developing/api/8/rest) to fetch the data from the Drupal backend and build the frontend in a completely separated and “independent” way. This way still leveraging the content management capabilities of Drupal which are great but losing a lot of other benefits that the framework provides out of the box.

An example of this approach is this website, where content is inserted into the Drupal backend and displayed using Angular2 and Angular Universal for server side rendering. But the frontend and the backend are completely decoupled, so if I wanted to build the website in React for example I wouldn’t need to change one line of code on the backend.


### Which one is better?

I don’t believe in absolute truth’s so I will not say one is better than the other, there is definitely value in both of them, and different situations demand different things when it comes to Team members, budget, time etc..

There has been a lot of discussion in the Drupal community [about how should Drupal be decoupled](http://buytaert.net/how-should-you-decouple-drupal) and I think there has been some progress in understanding both options but to try and find an absolute way in this is a bit of a pointless cause, just my 2 cents on that subject.

## Wrap up

This was just a small sample of the great effort both Drupal and Angular have been making in building bridges into other communities, and hopefully both keep on doing just that because that is what distinguishes them from other communities that tend to shut down and be a little more closed into cooperation and mixing up.

Some useful links on some of the things I talked in this post : 

*   [The future of decoupled drupal](http://buytaert.net/the-future-of-decoupled-drupal)
*   [New Orleans talk by Stephen Fluin](https://www.youtube.com/watch?v=PGneK_G5zhA&index=20&list=PLpeDXSh4nHjTY67K6N8RZHEiJhQL-dnUf)
*   [Angular 2 Drupal](https://github.com/manekinekko/angular2-drupal)
*   [Acquia Js Exploration](https://github.com/acquia/js-exploration/)
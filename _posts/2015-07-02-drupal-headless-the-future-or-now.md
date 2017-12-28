---
layout: post
title: "Drupal headless the future or now"
description: "After anouncement in Barcelona by @Dries Drupal 8 release candidate was released on October 7"
thumb_image: "posts/headless.jpeg"
tags: [web, drupal]
---

{% include image.html path="posts/headless.jpeg" path-detail="posts/headless.jpeg" alt="Drupal8 headless" %}

Drupal 8 is coming, and with it new aproaches to development websites with Drupal. One of my favourites as a front end developer is what has been called [Drupal Headless](https://github.com/davidhwang/horseman), meaning decoupling the Drupal backend from the front end leaving Drupal as only a content management doing what it does so good and leaving the front end to powerfull emerging tools like Angular, React or ember. 

## Why manage front end separately?

As a front end developer, one of the things (amongst others) that troubles me with Drupal is the fact that it leaves very few flexibility to how you manage your front end infrastructure. And having to manipulate front end through hooks and modules is not the most exciting thing to do in a world that is trully fascinating as front end is.

Another important aspect has to do with the fact that front end tools and frameworks moves much quickier that backend tools do, and this makes it very difficult for Drupal project (or any other for that matter) to keep up and keep in the same flow in terms of front end.

This approach leaves Drupal in peace leading with content and content workflow, permissions, roles, user management and all of the things that Drupal already does so well and leaves the front end to the front end developer that can now easily leverage from the power of frameworks like AngularJS, Backbone, Ember or any other that might come in the future.

## Drupal 8 Services

In Drupal 8 services are in core, which means by default Drupal has the capability to act as a powerfull content delivery system. With views you can make complex exports into json objects wich can be easily integrated into any front end, also authentication, users, entities, pretty much everything is there into the [REST module](https://www.drupal.org/documentation/modules/rest).

To me personally this is the most amazing thing that has happened in Drupal since fields came to the user profile in Drupal 7\. While managing a complex API in Drupal 7 is not a very easy process, in Drupal 8 this becomes out-of-the-box!

## What I am loosing with this approach?

So of course this has some implications and its not all roses in this approach. The most difficult thing to loose, and this is especially true for site builders (that will probably not take this approach anyway) is that you loose a lot of flexibility into managing your front end. As for now with views you can add fields, change views fields, add a panel here and there very easily, this of course does not exist and any change on the front end needs to be done by the front end developer. It will be just as easy to provide this field for example into the json output of the API but the front end developer needs to integrate this into the front end.

Another important thing that is lost is the capability of being able to change content and seeing live how that content is being displayed, even as a preview or with the content still unpublished untill it is perfect.

## Wrapping things up

We will definitely see more of this approac, and see Drupal sites with a whole new level of UX. While this is something that is happening is not yet a "thing" because the downsides for a large percentage of Drupal projects is still bigger than the advantages.

However for Drupal 8 it is now much easier to take this approach. If you want to take a look into some projects taking advantage of this approach take a look into the [Drupal headless group](https://groups.drupal.org/headless-drupal) where there are examples and constant converstation in the topic.
---
layout: post
title: "Rich sharing links in your angularJs headless drupal application"
description: "Learn how to enable rich sharing links in your AngularJs application."
thumb_image: "posts/image-angular-social.jpg"
tags: [web, drupal]
---

{% include image.html path="posts/image-angular-social.jpg" path-detail="posts/image-angular-social.jpg" alt="Angular social links" %}

Following on a [great post](http://www.michaelbromley.co.uk/blog/171/enable-rich-social-sharing-in-your-angularjs-app) by [Michael Browmley](https://twitter.com/michlbrmly) I decided to reshare this idea on how to enable rich sharing links in your AngularJs application.

The steps are really easy and enables you to have rich sharing links in you app with very little effort. I am going to focus on using this with Drupal rather than using a static php file like in the blog post from Michael.

## Open Graph and specific protocols

Facebook and all the major social media platforms expect your page to have some specific metatags to better display Images, description and other information about the link you are sharing. The most common from facebook is the OpenGraph protocol. it looks something like this :

{% highlight html %}
 <head>
    <meta property="og:title" content="Your own page title" />
    <meta property="og:description" content="A description of the page you are sharing." />
    <meta property="og:image" content="http://www.mysite.com/dist/images/nice_picture.jpg" />
 </head>
{% endhighlight %}

Twitter uses something simillar but with "twitter:" prefix instead of "og:". Once you submit a page to be shared the crawler from facebook or twitter is going to start scan your page to find these tags, and also scan the regular HTML to find some images or tags that it can use to better display the link.

## The problem with AngularJs

The problem with angular is that the page is rendered in Javascript and so there is no place where it can actually print a metatag with the correct information. Even if you do it dinamically via some service you would still render something like

{% highlight bash %}
 metadata.title 
 metadata.description
{% endhighlight %}

Because the crawler does not understand javascript and is expecting a immediate response of static content from the server, which he does not get he essencially cant resolve anything.

## The Solution - server side render for bot crawlers

Since you don't care how the page displays to these crawlers but rather just have the necessary tags and meta info that they need, you can use a server page to render this result to the crawlers. The only important thing is to make sure that only crawlers get this response and normal users are still getting the normal "Angular" page. Worth mentioning that this sever side page is [Drupal](https://www.drupal.org) which is giving us these pages essencially for free.

### We will be needing

*   A web server capable of URL rewriting. In this case, we are using Apache and the mod_rewrite module but we could use another option.
*   A server-side page to generate our crawler-friendly pages. Because we are using Drupal, and these are nodes we basically have this for free. We just point the crawler to the Drupal node page and voilla,&nbsp;Meta info and a nice server side page. We can also enhance it with custom modules&nbsp;for better meta data as well.
*   The Angular app must be using “html5mode” its URLs. This is because the # portion of a URL does not get sent to the server, so makes server-side redirection based on the Angular page impossible


## Apache Configuration

In this example I have my own blog in which URL's for specific pages match the ones in my blog_backoffice. So I have an angular page in&nbsp;http://joaogarin.com/blog/whats-coming-with-angular-2 and I simply point it to&nbsp;http://joaogarin.com/blog_backoffice/whats-coming-with-angular-2 which is the Drupal node for this page.

{% highlight bash %}
<ifModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTP_USER_AGENT} (facebookexternalhit/[0-9]|Twitterbot|Pinterest|Google.*snippet)
    RewriteRule ^blog/(.*)$ http://joaogarin.com/blog_backoffice/$1 [P]
</ifModule>
{% endhighlight %}

One you have this it will check the user agent for the correspondent crwalers, in this case Facebook, twitter, pinterest and google plus and it will redirect to the blog_backoffice directory where drupal is. You could essencially use anything here as long as you can map between them somehow. In my case its very easy since the URL's follow the same pattern.

## Testing it

You can now go ahead and test it out in each of the platforms.

*   [Facebook Open Graph Object Debugger](https://developers.facebook.com/tools/debug/)
*   [Twitter Card Validator](https://dev.twitter.com/docs/cards/validation/validator)
*   [Pinterest Rich Pin Validator](https://developers.pinterest.com/rich_pins/validator/)
*   [Google Structured Data Testing tool](http://www.google.com/webmasters/tools/richsnippets)

Enjoy your Angular App with Drupal backoffice.

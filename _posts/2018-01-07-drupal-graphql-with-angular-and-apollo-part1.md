---
layout: post
title: "Drupal GraphQL with Contenta, Angular and Apollo Part 1 - Authentication"
description: "Lets see how to get started using apollo drupal and angular."
thumb_image: "posts/net.jpg"
tags: [web, angular, javascript, drupal]
---

{% include image.html path="posts/net.jpg" path-detail="posts/net.jpg" alt="net" %}

Lately I have been experimenting a lot with different setups for working in decoupled arquitectures with drupal and javascript. During 2017 I have joined efforts with [Matt Davis](https://twitter.com/johnmattdavis) to create a [Angular app](https://github.com/contentacms/contenta_angular) for [Contenta](http://www.contentacms.org/) as one of the demos the distribution provides as default for new users, and also have been doing some side projects where I get to explore contenta and decoupled drupal in general.

One of my goals for 2018 was to get started exploring [GraphQL](http://graphql.org/), which fits together perfectly with decoupled drupal. Graphql is a query language by Facebook. It serves as a middleware between your client app and the server, making it super easy to understand as a frontend developer what you can query from the server, how responses are returned, caching strategies for fetching data and a lot more. For a more comprehensive description and feature set of Graphql visit the [Graphql site](http://graphql.org/).

In this post I want to focus on explaining how you can get started using [GraphQL](http://graphql.org/) with [Drupal](https://www.drupal.org/) using the [Drupal GraphQL module](https://github.com/drupal-graphql/graphql) and [Apollo Angular](https://github.com/apollographql/apollo-angular).

I will go through three of the most common things when it comes to building an app using these technologies together : 

-  Authentication
-  Queries
-  Mutations

So lets get started with part 1, Authentication!!

## Full code samples

The code samples can be found in Github. It includes all the frontend code and also the code for the custom mutations to be used with Drupal GraphQL.

[Frontend code](https://github.com/joaogarin/drupal-graphql-with-angular-and-apollo)
[Custom GraphQL mutation module](https://github.com/joaogarin/graphql_custom_mutations)

## Install Contenta

The first thing we need to do is to have an installation of [Contenta](http://www.contentacms.org/). There are informations on the page how to install but you can run the quick install by running the following command : 

{% highlight bash %}
php -r "readfile('https://raw.githubusercontent.com/contentacms/contenta_jsonapi/8.x-1.x/installer.sh');" > contentacms.sh
chmod a+x contentacms.sh
./contentacms.sh
{% endhighlight %}

One you have installed contenta, the Drupal graphql module will already be available for you to start using. It is an experimental module in contenta so take your cautions while using it altough it works pretty well and its already used in production by several companies.

### Graphql explorer

One of the awesome things graphql module brings is installing by default [GraphiQL](https://github.com/graphql/graphiql). Visit `http://yourwebsiteurl/graphql/explorer` and you will see GraphiQL there : 

{% include image.html path="posts/graphiql.png" path-detail="posts/graphiql.png" alt="graphiql" %}

GraphiQL is like a playground for you to start experimenting with GraphQL queries without having to setup anything. You can run queries, mutations, explore the API giving you all the information you need to experiment without having to setup a frontend. 

At this point you didnt have to configure or setup anything at all, and you have a working setup of Drupal and Graphql, ready to receive queries. Pretty cool hein?

Lets go deeper and start setting up a frontend and configure things for authentication, querying and mutations.

## Create your Angular frontend

The first thing we need to do in order to get started with our frontend is to create our angular app. Using the [Angular CLI](https://github.com/angular/angular-cli) this is essencially just running : 

{% highlight javascript %}
ng new PROJECT-NAME
{% endhighlight %}

And we have our angular app. The next step is to setup authentication using drupal [simple auth module](https://www.drupal.org/project/simple_oauth) and [OAuth 2 and OpenId Connect (OIDC)](https://github.com/manfredsteyer/angular-oauth2-oidc).

## Authentication

Because Drupal's [simple auth module](https://www.drupal.org/project/simple_oauth) is built around Oauth2 we can easily leverage libraries to make it easier to handle authentication on the frontend, in this example I will be using [OAuth 2 and OpenId Connect (OIDC)](https://github.com/manfredsteyer/angular-oauth2-oidc) but you could easily use any other or even none and do it all by yourself.

Another great thing about [GraphQL](http://graphql.org/) is that using authentication works exactly like it does in any HTTP-like environment, using common strategies and standars as for example oauth2.

### Configuring simple auth

The first thing we need to do with Contenta in order to get authentication working is setting up the simple auth module to handle authentication. Because [Mateu Aguiló Bosch](https://twitter.com/e0ipso) has such [great tutorials](https://www.youtube.com/watch?v=rTcC0maPLSA&list=PLZOQ_ZMpYrZtqy5-o7KoDhM3n6M0duBjX) about how to setup simple auth (all available in Contenta's tutorials btw) I wont go through this here to save some time in the blog post. All I can say is I just followed it step by step and it worked great. For this post and in the case I have used I was using [implicit grant](https://www.youtube.com/watch?v=sZbzcCXMmEA&list=PLZOQ_ZMpYrZtqy5-o7KoDhM3n6M0duBjX&index=9) as the method for authentication.

There are howeve a couple things I found could go a bit more "wrong" when setting this up so make sure to pay attention to : 

-  Private and public key's permissions (warnings will be give in reports if configured wrong)
-  make sure the role for the user has permissions to simple auth extras ("Allow using the AuthCode grant")

At this point authentication is setup on the server (Drupal) and we can move on to configuring it on the client.

### Configuring Angular Oauth2

For connecting Angular to our Drupal backend we will be using [OAuth 2 and OpenId Connect (OIDC)](https://github.com/manfredsteyer/angular-oauth2-oidc).

The first thing to be done is import the NgModule provided by the package in our application. In your app.module.ts file (in src/app) add the NgModule : 

{% highlight javascript %}
import { OAuthModule } from 'angular-oauth2-oidc';
[...]

@NgModule({
  imports: [ 
    [...]
    HttpModule,
    OAuthModule.forRoot()
  ],
  declarations: [
    AppComponent,
    HomeComponent,
    [...]
  ],
  bootstrap: [
    AppComponent 
  ]
})
export class AppModule {
}
{% endhighlight %}

Next we will follow the [instructions](https://github.com/manfredsteyer/angular-oauth2-oidc#configuring-for-implicit-flow) to configure the module for implicit flow by providing a auth.config.ts file that the package will use. this file can be located in the src/app/ folder.

{% highlight javascript %}
import { environment } from './../environments/environment';
import { AuthConfig } from 'angular-oauth2-oidc';

export const authConfig: AuthConfig = {

  // Url of the Identity Provider
  loginUrl: `${environment.contenta_url}/oauth/authorize`,

  // URL of the SPA to redirect the user to after login
  redirectUri: 'http://localhost:4200/',

  // The SPA's id. The SPA is registerd with this id at the auth-server
  clientId: 'YOUR_CLIENT_ID_PROVIDED_IN_SIMPLE_AUTH_CLIENT',

  // set the scope for the permissions the client should request
  // The first three are defined by OIDC. The 4th is a usecase-specific one
  scope: 'content_administrator',
  oidc: false,
  strictDiscoveryDocumentValidation: false,
  showDebugInformation: true,
}
{% endhighlight %}

A couple of things to note here : 

First we provide the url of contenta so that when logging in you are redirected to that url in order to authenticate. 

You also need to provide a redirectUrl which is where the user will be redirected once login ends, we provide the default location of our app (if you are using a different url or port make sure to change this). 

The last important thing to notice here is the scope, if you have watched the videos mentioned above on how to setup simple_auth you have [seen that you must create roles](https://www.youtube.com/watch?v=-xRBJvmg0qQ&list=PLZOQ_ZMpYrZtqy5-o7KoDhM3n6M0duBjX&index=4), which will give simple auth the necessary information about a user's roles and what he has access to.

The configuration has a couple of extra options you can use so make sure to refer to the [official documentation](https://github.com/manfredsteyer/angular-oauth2-oidc) for more information on all the possible options.

Now we need to connect the config with the app in our app's root component (app.component.ts) : 

{% highlight javascript %}
import { Component } from '@angular/core';
import { OAuthService } from 'angular-oauth2-oidc';
import { JwksValidationHandler } from 'angular-oauth2-oidc';
import { authConfig } from './auth.config';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';
  authenticated = false;

  constructor(private oauthService: OAuthService, public apollo: Apollo) {
    this.configureWithNewConfigApi();
  }

  private configureWithNewConfigApi() {
    this.oauthService.configure(authConfig);
    this.oauthService.setStorage(sessionStorage);
    this.oauthService.tryLogin().then(_ => {
      this.authenticated = this.oauthService.hasValidAccessToken();
    });
  }

  public login() {
    this.oauthService.initImplicitFlow();
  }

  public logoff() {
    this.oauthService.logOut();
    this.authenticated = false;
  }
}
{% endhighlight %}

Now we just need our template to call the login and logout methods : 

{% highlight html %}
<h1>
  Hello Contenta Graphql
</h1>

<div *ngIf="authenticated;else unauth">
  <h2>Status : Authenticated</h2>
  <div>
    <button *ngIf="authenticated" class="btn btn-default" (click)="logoff()">
      Logout
    </button>
  </div>
</div>
<ng-template #unauth>
  <h2>Status : Anonymous</h2>
  <div>
    <button *ngIf="!authenticated" class="btn btn-default" (click)="login()">
      Login
    </button>
  </div>
</ng-template>
{% endhighlight %}

Thats it!! At this point authentication is working. You should be able to run the app by running : 

{% highlight bash %}
npm start
{% endhighlight %}

and click the button "login" in order to get redirected to contenta, provide your authentication credentials and get redirected back to the app. The token provided by simple auth is then registered in the oauth package we used and you can access it inside your app by doing : 

{% highlight javascript %}
this.oauthService.getAccessToken()
{% endhighlight %}

In the [next post](/posts/drupal-graphql-with-angular-and-apollo-part2) we  will see how you can connect Apollo and GraphQL to our authentication in order to provide authorization / controlled access for Queries and Mutations.
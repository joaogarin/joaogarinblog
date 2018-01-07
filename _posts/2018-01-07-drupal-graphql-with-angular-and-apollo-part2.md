---
layout: post
title: "Drupal GraphQL with Contenta, Angular and Apollo Part 2 - Queries"
description: "Using Apollo angular to query entities from Contenta."
thumb_image: "posts/net.jpg"
tags: [web, angular, javascript, drupal]
---

{% include image.html path="posts/queries.jpg" path-detail="posts/queries.jpg" alt="queries" %}

This is the continuation of a series of blog posts explaining how to use [Contenta](http://www.contentacms.org/) and [GraphQL](http://graphql.org/) with [Angular](https://angular.io/).

In the [previous post](http://joaogarin.com/posts/drupal-graphql-with-angular-and-apollo-part1) we learned how to configure Authentication, and now we will see how to use that authentication to query and mutate data in our GraphQL backend using [Apollo Angular](https://github.com/apollographql/apollo-angular).

## Install Apollo Angular

The first step is to install Apollo Angular. Following the [Instructions on the offical docs site](https://www.apollographql.com/docs/angular/) we start by installing it via npm : 

{% highlight bash %}
npm install apollo-angular apollo-angular-link-http apollo-client apollo-cache-inmemory graphql-tag graphql --save
{% endhighlight %}

Next we import the ApolloModule into our app's root module (app.module.ts) and configure the Apollo Client. The Client is what we will be using to call queries, mutations and so on to our graphql backend. Because we will be setting up our client to use authentication we need to configure Apollo Client with a specific link (Apollo Link) that will append the token we receive from Drupal (see previous authentication blog post) to all of the requests being made. You can find more information about Authentication in the [official documentation for Apollo Angular](https://www.apollographql.com/docs/angular/recipes/authentication.html)

{% highlight javascript %}
import { Apollo, ApolloModule } from 'apollo-angular';
import { HttpLink, HttpLinkModule } from 'apollo-angular-link-http';
import { setContext } from 'apollo-link-context';
import { InMemoryCache } from 'apollo-cache-inmemory';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    OAuthModule.forRoot(),
    ApolloModule,
    HttpLinkModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(apollo: Apollo,
    httpLink: HttpLink) {
    const http = httpLink.create({ uri: `${environment.contenta_url}${environment.graphql}`});

    const auth = setContext((_, { headers }) => {
      // get the authentication token from local storage if it exists
      const token = localStorage.getItem('token');

      if (!token) {
        return {};
      } else {
        return {
          headers: new HttpHeaders().append('Authorization', `Bearer ${token}`).append('Content-Type', 'application/json')
        };
      }
    });

    apollo.create({
      link: auth.concat(http),
      cache: new InMemoryCache(),
    });
  }
}
{% endhighlight %}

There are a couple of things going on here, lets break this down into pieces :

We configure our http link by providing our backend api URL (contenta graphql endpoint). We then create a authentication context that will append our token to each http request going out with the HTTP header "Authorization" bearer and proving our token (previously saved in localStorage).

### Saving our token

In our previous blog post we had a little snippet to kickstart our authentication, we need to adapt this to save our token in localStorage so that it is available for us here. In our app.component.ts change the method "configureWithNewConfigApi" to : 

{% highlight javascript %}
private configureWithNewConfigApi() {
  this.oauthService.configure(authConfig);
  this.oauthService.setStorage(sessionStorage);
  this.oauthService.tryLogin().then(_ => {
    // save the token in localstorage
    localStorage.setItem('token', this.oauthService.getAccessToken());
    this.authenticated = this.oauthService.hasValidAccessToken();
  });
}
{% endhighlight %}

We also need to make sure to dispose it when we logout, so adapt the logout() method with : 

{% highlight javascript %}
public logoff() {
  this.oauthService.logOut();
  localStorage.removeItem('token');
  this.authenticated = false;
  // reset the store after that
  this.apollo.getClient().resetStore();
}
{% endhighlight %}

ok! At this point Apollo is fully configured, we can start making queries. Also notice that by doing it this way we need to make sure that the roles that we are using (see previous blog post about Authentication) have the needed access rights to read / write or whatever you need to do to the correct entities.

## Querying

Querying with Apollo is extremely easy, I suggest getting started by playing with the GraphiQL playground and experiment with queries structure / syntax first in order to make sure the queries work there before doing it on the frontend.

For this example I will assume a content type called "Client" exists in the Drupal installation (just setup a content type with name Client, machine name should be client). Also this content type should have some fields created : 

-  email
-  telephone

### Querying a list of clients

Before I do any frontend work (like mentioned above) I tested my query in GraphiQL Playground, thats good because it provides autocomplete for fields that are available in any entity, relationships etc..Once that's done I can move to the frontend app.

{% include image.html path="posts/graphiql.png" path-detail="posts/graphiql.png" alt="graphiql" %}

Now that the query works in GraphiQl the first step we can do to start working with clients in the frontend is creating a model for Clients. Inside src/app/ create a folder "models" with a file called client.model.ts : 

{% highlight javascript %}
export interface Client {
  entityLabel: string;
  email: string;
  telephone: string;
}
{% endhighlight %}

Now in the app.component.ts (our root component for simplicity sake) lets do a simple query to fetch clients. Our query will return an observable of our response (a list of clients) we can then pipe this in our template using the `async` pipe.

{% highlight javascript %}
...

import { Observable } from 'rxjs/observable';

import { Apollo } from 'apollo-angular';
import gql from 'graphql-tag';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';
  authenticated = false;
  clients: Observable<Client[]>;

  query = gql`{
  nodeQuery(limit: 5, offset: 0, filter: {type: "Client"}) {
    entities {
      entityLabel
      entityUrl {
        path
      }
      entityPublished
      ... on NodeClient {
        telephone,
        email
      }
    }
  }
}`;

  ...

  constructor(private oauthService: OAuthService, public apollo: Apollo) {
    this.configureWithNewConfigApi();
  }

  ...

  runQuery() {
    // Sample query
    this.clients = this.apollo .query({ query: this.query}).map(q => {
      return q.data['nodeQuery']['entities'];
    });
  }
  ...
}

{% endhighlight %}

In our template we append a little button to run the Query : 

{% highlight html %}
...
<button (click)="runQuery()">query</button>
...
}
{% endhighlight %}

We query our data using Apollo Client and wrapping our graphql in graphql-tag. The query is exactly the same we used in the GraphiQL playground. We map the response and get the entities our of the data received and assign it to our observable of type `clients: Observable<Client[]>;`.

In our template we then can easily list these out : 

{% highlight html %}
...
<ul>
  <li *ngFor="let client of clients | async">
    <div>
      Name: {{client.entityLabel}}
    </div>
    <div>
      Email: {{client.email}}
    </div>
    <div>
      Telephone: {{client.telephone}}
    </div>
  </li>
</ul>
...
}
{% endhighlight %}

## All together now

The full app.module.ts with authentication and apollo client and link configured looks like this : 

{% highlight javascript %}
import { environment } from './../environments/environment';
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { HttpClientModule, HttpHeaders } from '@angular/common/http';
import { AppComponent } from './app.component';
import { OAuthModule } from 'angular-oauth2-oidc';

import { Apollo, ApolloModule } from 'apollo-angular';
import { HttpLink, HttpLinkModule } from 'apollo-angular-link-http';
import { setContext } from 'apollo-link-context';
import { InMemoryCache } from 'apollo-cache-inmemory';

@NgModule({
  declarations: [
    AppComponent,
  ],
  imports: [
    BrowserModule,
    HttpClientModule,
    OAuthModule.forRoot(),
    ApolloModule,
    HttpLinkModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule {
  constructor(apollo: Apollo,
    httpLink: HttpLink) {
    const http = httpLink.create({ uri: `${environment.contenta_url}${environment.graphql}` //?XDEBUG_SESSION_START=PHPSTORM 
   });

    const auth = setContext((_, { headers }) => {
      // get the authentication token from local storage if it exists
      const token = localStorage.getItem('token');

      if (!token) {
        return {};
      } else {
        return {
          headers: new HttpHeaders().append('Authorization', `Bearer ${token}`).append('Content-Type', 'application/json')
        };
      }
    });

    apollo.create({
      link: auth.concat(http),
      cache: new InMemoryCache(),
    });
  }
}
{% endhighlight %}

Our root app.component.ts where we have our query for clients : 

{% highlight javascript %}
import { Client } from './models/Client';
import { Component } from '@angular/core';
import { OAuthService } from 'angular-oauth2-oidc';
import { JwksValidationHandler } from 'angular-oauth2-oidc';
import { authConfig } from './auth.config';

import { Observable } from 'rxjs/observable';

import { Apollo } from 'apollo-angular';
import gql from 'graphql-tag';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = 'app';
  authenticated = false;
  clients: Observable<Client[]>;

  query = gql`{
  nodeQuery(limit: 5, offset: 0, filter: {type: "Client"}) {
    entities {
      entityLabel
      entityUrl {
        path
      }
      entityPublished
      ... on NodeClient {
        telephone,
        email
      }
    }
  }
}`;

  constructor(private oauthService: OAuthService, public apollo: Apollo) {
    this.configureWithNewConfigApi();
  }

  runQuery() {
    // Sample query
    this.clients = this.apollo .query({ query: this.query}).map(q => {
      return q.data['nodeQuery']['entities'];
    });
  }

  private configureWithNewConfigApi() {
    this.oauthService.configure(authConfig);
    this.oauthService.setStorage(sessionStorage);
    this.oauthService.tryLogin().then(_ => {
      // save the token in localstorage
      localStorage.setItem('token', this.oauthService.getAccessToken());
      this.authenticated = this.oauthService.hasValidAccessToken();
    });
  }

  public login() {
    this.oauthService.initImplicitFlow();
  }

  public logoff() {
    this.oauthService.logOut();
    localStorage.removeItem('token');
    this.authenticated = false;
    // reset the store after that
    this.apollo.getClient().resetStore();
  }
}
{% endhighlight %}

and our template for this component where we render the full list of clients we get from graphql : 

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

<h2>
  Clients
</h2>

<button (click)="runQuery()">query</button>

<ul>
  <li *ngFor="let client of clients | async">
    <div>
      Name: {{client.entityLabel}}
    </div>
    <div>
      Email: {{client.email}}
    </div>
    <div>
      Telephone: {{client.telephone}}
    </div>
  </li>
</ul>
{% endhighlight %}

Thats it! Thats all there is too queries with graphql and contenta using simple auth as the authentication mechanism. You can find all the things you can do with queries including creating fragments for shared parts of queries across multiple places of you app and other more advanced query features in the [Official documentation site](https://www.apollographql.com/docs/angular/).

In the next post we will see how we can do mutations using the [Drupal GraphQL module](https://github.com/drupal-graphql/graphql) and Apollo.
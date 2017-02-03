---
title: Initialization
order: 2
---
<h2 id="installation">Installation</h2>

Install the `apollo-client` npm package, the `apollo-angular` integration package, and the `graphql-tag` library for constructing query documents:

```bash
npm install apollo-client apollo-angular graphql-tag --save
```

> If you are in an environment that does not have a global [`fetch`](https://developer.mozilla.org/en-US/docs/Web/API/GlobalFetch/fetch) > implementation, make sure to install a polyfill like [`whatwg-fetch`](https://www.npmjs.com/package/whatwg-fetch).

<h2 id="initialization">Initialization</h2>

Add Angular Apollo into your project by creating an `ApolloClient` and use `ApolloModule`. `ApolloClient` serves as a central store of query result data which caches and distributes the results of our queries. `ApolloModule` wires that client into your application.

Angular Apollo client can be created as a simple object or as a class. Creating a class allows to use other providers, such as configuration service for the url. 

<h3 id="creating-object-client">Creating an object client</h3>

To get started, create an [`ApolloClient`](/core/apollo-client-api.html#constructor) instance and point it at your GraphQL server:

```ts
import { ApolloClient } from 'apollo-client';

// by default, this client will send queries to `/graphql` (relative to the URL of your app)
const client = new ApolloClient();
```

The client takes a variety of [options](/core/apollo-client-api.html#constructor), but in particular, if you want to change the URL of the GraphQL server, you can pass in a custom [`NetworkInterface`](/core/apollo-client-api.html#NetworkInterface):

```ts
import { ApolloClient, createNetworkInterface } from 'apollo-client';

// by default, this client will send queries to `/graphql` (relative to the URL of your app)
const client = new ApolloClient({
  networkInterface: createNetworkInterface({
    uri: 'http://my-api.grapql.com'
  }),
});

export function getClient(): ApolloClient {
  return client;
}
```

The other options control the behavior of the client, and we'll see examples of their use throughout this guide.

Use the ApolloModule `forRoot` method to provide the client object. 
In the `app.module.ts`: 

```ts
import { ApolloClient } from 'apollo-client';
import { ApolloModule } from 'apollo-angular';

import { NgModule } from '@angular/core';
import { BrowserModule  } from '@angular/platform-browser';
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic';

import { AppComponent } from './app.component';
import { getClient } from './client';

@NgModule({
  imports: [
    BrowserModule,
    ApolloModule.forRoot(getClient)
  ],
  declarations: [ AppComponent ],
  bootstrap: [ AppComponent ]
})
class AppModule {}

platformBrowserDynamic().bootstrapModule(AppModule);
```

> There is still `ApolloModule.withClient` available but we recommend you to use `ApolloModule.forRoot` instead.

<h3 id="creating-class-client">Creating a class client</h3>

Create a client Injectable provider. You can use services in the provider. 
This re-exports an ApolloModule with the class client. 
```ts
import { ApolloClient, createNetworkInterface } from 'apollo-client';
import { Injectable, NgModule } from '@angular/core';
import { Apollo, ApolloModule, APOLLO_DIRECTIVES } from 'apollo-angular';
import { ConfigService } from './config.service'; //Any configuration service(s)

@Injectable()
export class Client {
  public client: ApolloClient;
  private networkInterface: any;

  constructor(private config: ConfigService) { //inject configuration service
    this.networkInterface = createNetworkInterface({
      uri: this.config.getGraphQLUrl(),
      opts: {
        credentials: 'same-origin',
      },
    });
   
    this.client = new ApolloClient({
      networkInterface: this.networkInterface
    });
  }
}

export function ApolloFactory(client: Client) {
  return new Apollo({ 'default': client.client });
}

@NgModule({
  providers: [Client, {
    provide: ApolloModule,
    useFactory: ApolloFactory,
    deps: [Client]
  }]
})

export class ConfiguredApolloModule { }
```
Then in the `app.module.ts`: 
```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule, ReactiveFormsModule } from '@angular/forms';
import { ApolloModule } from 'apollo-angular';

import { ConfigService } from './config.service';
import { AppComponent } from './app.component';
import { ConfiguredApolloModule } from './client';

@NgModule({
  declarations: [
    AppComponent,
  ],
  providers: [ConfigService],
  imports: [
    BrowserModule,
    FormsModule,
    ReactiveFormsModule,
    // Define the revised ApolloClient
    ConfiguredApolloModule,
  ],
  bootstrap: [AppComponent],
})
export class AppModule { }
```


<!--  Add content here once it exists -->
<!-- ## Troubleshooting -->

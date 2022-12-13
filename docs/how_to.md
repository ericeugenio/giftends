# How To

By Gil Dobrovinsky, Roos Madelief and Eric Eugenio.

# Table of contents

- [Introduction](#introduction)
- [Exposure](#exposing)
    - [CDN](#cdn)
    - [FTP Server](#ftp-server)
    - [GraphQL Server](#graphql-server)
        - [Basics](#basics)
        - [Subscriptions](#bonus-subscriptions)
    - [WS Server](#ws-server)
    - [REST Server](#rest-server)
        - [Firebase](#firebase)
        - [Endpoints](#endpoints)
    - [Storage Server](#storage-server)
- [Integration](#integration)

# Introduction

In this document we will explain how the different services of the system were implemented.

See [choices](/choices.md) and [overview_of_the_system](/overview_of_the_system.md) for further explanation on the system architecture.

# Exposure

## CDN

To expose the logo of our system we are using **Azure CDN profile**. However, we also need a palce to storage the distributed image, therefore we also make use of **Azure Storage**.

This are the steps followed to store the image in Azure:

1. Create an Azure Storage account.
2. Create a container inside Azure Storage Acount with "Blob" access level, we do not want users to be authenticated in order to acces the image CDN.
3. Upload the image to the container.

Now that the image is stored we want to deliver it through the CDN:

1. From the Storage account, go to the "Azure CDN" tab from the "Security + Networking" menu.
2. Create a new endpoint from a new CDN profile

We can now acces our image thorugh the CDN by using the format:

```
https://<endpoint-name>.azureedge.net/<container-name>/<image>
```

Our logo can then be retrieved though the following link: <https://giftends.azureedge.net/media/giftends-logo.png>

## FTP Server

FTP Server is hosted in an **Azure Virtual Machine**. Bellow are the steps to configure the FTP server in the VM:

1. Create an Azure Virtual Machine with Filezilla FTP server.
2. Remotely connect to VM.
3. Configure FTP server properties.
4. Define users and groups on FTP server and assign home directories with proper permissions.
5. Test server with FTP client.

> **TODO:** to be implemented...

## GraphQL Server 

As explained in [choices](/choices.md) the GraphQL server is hosted in **Azure App Service**. However, this time, instead of using the **Azure Portal** to create the app, we have used Visual Studio Code and **Azure Extensions**.

Rather than focusing on how the **Azure App Service** was created and how we deploy it, we are going to explain how the GraphQL Server is implemented and exposed.

> **Bonus:** subscriptions have been implemented.

### Basics

Server has been implemented using Node with express and [graphql](https://www.npmjs.com/package/graphql) and [express-graphql](https://www.npmjs.com/package/express-graphql) libraries.

A GraphQL server requires a schema made of queries, mutations, and subscriptions. Our system requires products to be read but not modified. Thus mutations are not implemented, and subscriptions are explained in the [next section](#bonus-subscriptions).

Before diving into the quieries implementation, we need to declare the server's types, as we will be exposing products, the following type is defined according to the partner's group db:

```js
const ProductType = new graphql.GraphQLObjectType({
    name: 'Product',
    fields: () => ({
        id: { type: graphql.GraphQLID },
        name: { type: graphql.GraphQLString },
        description: { type: graphql.GraphQLString },
        link: { type: graphql.GraphQLString },
        price: { type: graphql.GraphQLFloat },
        rating: { type: graphql.GraphQLFloat },
        category: { type: graphql.GraphQLString },
        image: { type: graphql.GraphQLString },       
    })
})
```

After defining the *ProductType* we defined two different sample queries to retireve products. A first one to retrieve all products and a second one to retreive a product by its id.

```js
const QueryType = new graphql.GraphQLObjectType({
    name: 'Query',
    fields: {
        products,
        productById
    }
})
```

Where *products* and *productById* are:

```js
const products = {
    type: new graphql.GraphQLList(ProductType),
    resolve: (root, args, {db}, info) => {
        return db.getProducts()
    }
}

const productById = {
    type: ProductType,
    args: {
        id: { type: new graphql.GraphQLNonNull(graphql.GraphQLID) }
    },
    resolve: (root, {id}, {db}, info) => {
        return db.getProductById(id)
    }
}
```

Notice how products are retrieved from a db connector stored in the GraphQL instance.

With the *QueryType* defined, we can noe define our schema (we will add subscriptions later on):

```js
const schema = new graphql.GraphQLSchema({
    query: QueryType
})
```

Finally, with our schema defined, we can add a route into our express server to host the GraphQL:

```js
const app = express()
const port = process.env.PORT || 8080
const context = { db: db }

app.use(cors())
app.use(express.json())
app.use(express.urlencoded({ extended: true }))

app.use('/graphql', graphqlHTTP({
    schema: schema,
    context: context,
    graphiql: true
}))

app.listen(port, (err) => {
    if (err) {
        throw err
    }
})
```

### **Bonus**: Subscriptions

GraphQL subscriptions, allow clients to be notified by the server of specific events. For Server-Client communication we implemented a WebSocket Server with the Publisher-Subscriber pattern. Thus, the following libraries are added to the project: [ws](https://www.npmjs.com/package/ws), [graphql-ws](https://www.npmjs.com/package/graphql-ws?activeTab=readme), and [graphql-subscriptions](https://www.npmjs.com/package/graphql-subscriptions).

Similarly to queries, we started by defining subscription topics. Our server only supports reading products, therefore, we decided to notify users of the products queried by id. Thus clients can be aware of the last products viewed by other clients (i.e. trending products). The subscription query in node looks like this:

```js
const SubscriptionType = new graphql.GraphQLObjectType({
    name: 'Subscription',
    fields: {
        lastProduct
    }
})
```

Where *lastProduct* is:

```js
const lastProduct = {
    type: ProductType,
    subscribe: (root, args, {pubsub}, info) => {
        return pubsub.asyncIterator('LAST_PRODUCT_READ')
    },
    resolve: (payload) => payload.product
}
```

Notice how this time we retrieve a PubSub insatnce from the context, as we did with the db, this allow us to share the same instance across the GraphQL server. Having our subscription object ready we now have to update our schema:

```js
const schema = new graphql.GraphQLSchema({
    query: QueryType,
    subscription: SubscriptionType
})
```

Before implementing the WebSocket server, we need the server to send events when a product is retrieved by id, thus we modified the *productById* query as such:

```js
const productById = {
    type: ProductType,
    args: {
        id: { type: new graphql.GraphQLNonNull(graphql.GraphQLID) }
    },
    resolve: (root, {id}, {db, pubsub}, info) => {
        var product_promise = db.getProduct(id)
        
        /* Notify of viewd products */
        pubsub.publish('LAST_PRODUCT_READ', {
            product: product_promise
        })

        return product_promise
    }
}
```

Now, every time the *productById* query is triggered the server will inform the sbscribed clients about the queried product. Finally, is time to set up the WS server, by modifying a few lines in the express server:

```js
...

const context = {
    db: db,
    pubsub: new PubSub(),
}

...

const httpServer = app.listen(port, (err) => {
    if (err) {
        throw err
    }
    else {
        const wsServer = new WebSocketServer({
            server: httpServer,
            path: '/graphql/subscriptions'
        })

        const wsOptions { 
            schema, 
            context: context 
        }
        
        useServer(wsOptions, wsServer)
    }
})
```

If the GraphQL server runs without problems, we start the WS server. 

## WS Server

> **TODO:** to be implemented...

## REST Server 

Authentication is done through a REST API hosted in an **Azure Function App** containing three different functions/endpoints triggered by an HTTP Request:

- [/signup](#signup)
- [/login](#login)
- [/acceptInvite](#acceptinvite)

As with the GraphQL server, instead of using the Azure Portal to create the function app, we have used Visual Studio Code and Azure Extensions. Thus, we are only going to focus how the Authentication API is implemented and exposed.

Authentication is done through node and Firebase, so before explaining the API implementation we will explain how to set up a firebase project in node.

### Firebase

Follow the steps bellow to configure a Firebase project in Node:

1. Login in the [Firebase](https://firebase.google.com/) website.
2. Add a project.
3. Add authentication to the project (through email and password in our case).
4. Create a web app for the project.
5. Copy the given config in JSON format.

Having Firebase set up, we can now start the node project and init firebase, for it we are going to use the [firebase](https://www.npmjs.com/package/firebase?authuser=0) package for node.

With the package installed we can init our Firebase app with the following code:

```js
import { initializeApp } from "firebase/app"
import { getAuth } from "firebase/auth"

/* Copied config from Firebase Console*/
const config = {...}

const app = initializeApp(config)
const auth = getAuth(app)
```

### Endpoints

The exposed API allows users to signup, login, and accept friend invites from other users. In **Azure Function apps** endpoints are represented by **Azure functions** thus our app has the following directory tree structure:

```bash
├── auth_acceptInvite
│   ├── function.json
│   └── index.js
├── auth_login
│   ├── function.json
│   └── index.js
├── auth_signup
│   ├── function.json
│   └── index.js
├── host.json
├── local.settings.json
├── package-lock.json
└── package.json
```

Notice each function has it's own configuration file (*function.json*). We are only going to update two fileds: "route" and "methods". And from the *host.json* configuration file we are going to add an empty string to the "routePrefix" field to delete the "/api" route added by default.

#### /signup

Signup "route" is set to "/auth/signup" and "methods" to only POST. Implementation, relies on Firebase module using the function:

```js
import { createUserWithEmailAndPassword } from firebase/auth"
```

The function takes an email and a password as a parameter and creates a user and signs in it on success. Function can fail due to exisiting email or invalid email or password. 

On succes, we return the user, as well as a JWT token and a refresh token, also the expiration time of the token. On failure we report the error.

#### /login

Similarly to signup, login "route" is set to "/auth/login" and "methods" to only POST. Implementation, relies on Firebase module using the function:

```js
import { signInWithEmailAndPassword } from firebase/auth"
```

The functions behaves as the previous one, but instead of creating the user with the entered credentials, it checks from the existing users to sign in.

On succes, we return the user, as well as a JWT token and a refresh token, also the expiration time of the token. On failure we report the error.

#### /acceptInvite

> **TODO:** to be implemented...

## Storage Server 

> **TODO:** to be implemented...

# Integration

> **TODO:** to be implemented...


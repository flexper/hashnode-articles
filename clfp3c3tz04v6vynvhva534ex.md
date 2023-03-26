---
title: "unify-errors : Unify all errors across protocols and libs"
datePublished: Sun Mar 26 2023 07:40:39 GMT+0000 (Coordinated Universal Time)
cuid: clfp3c3tz04v6vynvhva534ex
slug: unify-errors-unify-all-errors-across-protocols-and-libs
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1678896419851/17f7bc57-97bb-4e66-b462-b03ef5b58ba8.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1678898894985/c019ffce-e680-4192-b4db-d153bcd44aef.png
tags: error-handling, typescript, fastify, mercurius, unify-errors

---

# Presentation

Properly handling errors is crucial for building reliable and stable applications.

[unify-errors](https://www.npmjs.com/package/unify-errors) is a TypeScript library that provides a **simple and consistent** way to define and handle errors across your NodeJS HTTP server.

It includes a set of **pre-defined error types**, and also allows you to define **your own custom errors**. With Unify-Errors, you can ensure that all errors in your application are consistent and easy to handle.

In this article, we'll talk about how it works and how we, at [Flexper](https://flexper.hashnode.dev/), are using it to normalize error handling within our stack, with **Fastify** using [unify-fastify](https://github.com/flexper/unify-fastify) and in our more **GraphQL**\-specific use cases, with [unify-mercurius](https://github.com/flexper/unify-mercurius), both of them implementing unify-errors.

# Use

## Install

```bash
npm install unify-errors
```

## API

To use Unify-Errors in your application, simply import the error type that you need and throw it when an error occurs. Here is an example:

```typescript
import { BadRequest } from 'unify-errors';

function errorExample() {
    throw new BadRequest({
        context: "Example context"
    });
}
```

The library exposes all the basic HTTP errors:

* BadRequest
    
* Forbidden
    
* InternalSeverError
    
* NotFound
    
* NotImplemented
    
* TimeOut
    
* Unauthorized
    

You can create your very own errors too!

```typescript
import { CustomError, CustomErrorContext } from 'unify-errors';

export class MyCustomError extends CustomError {
  constructor(public context?: CustomErrorContext) {
    super('My custom error was thrown !', context);

    // Set the prototype explicitly.
    Object.setPrototypeOf(this, MyCustomError.prototype);
  }
}
```

# Adapters

As unify-errors wraps all the high level requirements, we decided to make some adapters to easily use them inside our stack

## [unify-fastify](https://github.com/flexper/unify-fastify)

From the several NodeJS server providers, we decided to go from NestJS to Fastify.

In order to help us across it, we've made a Fastify Plugin that can throw the errors exposed with unify-errors

### Install

```bash
npm i unify-fastify
```

### API

Here is a simple *Hello World* example to create a Fastify server that can throw a basic HTTP error: **Bad Request**

```typescript
import fastify from 'fastify'
import unifyFastifyPlugin from 'unify-fastify';
import { BadRequest } from 'unify-errors';

const server = fastify()
server.register(unifyFastifyPlugin, { /* options */ })

server.get('/bad-request', async () => {
  throw new BadRequest({ example: 'A bad request error'})
})
```

### Plugin options

| name | default | description |
| --- | --- | --- |
| *hideError* | false | Return stack or not |

## [unify-mercurius](https://github.com/flexper/unify-mercurius)

GraphQL is a marvelous way to exchange complex structured data and probably one of the best providers to use with Fastify is Mercurius.

So once again, we've made an error formater for Mercurius to handle errors thrown by unify-errors

### Install

```bash
npm i unify-mercurius
```

### API

Here is how you would use unify-mercurius as an error formatter in a basic Fastify with Mercurius project

```javascript
'use strict'

const Fastify = require('fastify')
const mercurius = require('mercurius')
const { unifyMercuriusErrorFormatter } = require('unify-mercurius')

const app = Fastify()

const schema = `
  type Query {
    add(x: Int, y: Int): Int
  }
`

const resolvers = {
  Query: {
    add: async (_, { x, y }) => x + y
  }
}

app.register(mercurius, {
  schema,
  resolvers,
  errorFormatter: unifyMercuriusErrorFormatter()
})

app.get('/', async function (req, reply) {
  const query = '{ add(x: 2, y: 2) }'
  return reply.graphql(query)
})

app.listen(3000)
```

# Conclusion

unify-errors is a simple and useful TypeScript library for handling errors in your application. By providing a consistent and easy-to-use API, it helps to ensure that all errors are properly handled and that your application remains reliable and stable.
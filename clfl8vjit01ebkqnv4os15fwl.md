---
title: "Effortlessly paginate your Prisma ORM queries with paginate-prisma"
datePublished: Thu Mar 23 2023 15:04:39 GMT+0000 (Coordinated Universal Time)
cuid: clfl8vjit01ebkqnv4os15fwl
slug: effortlessly-paginate-your-prisma-orm-queries-with-paginate-prisma
tags: nodejs, apis, graphql, rest-api, prisma

---

## Introduction

When working with Prisma ORM, it's common to need pagination functionality to retrieve and display data in smaller, manageable chunks. Pagination is a technique that helps split a large amount of data into smaller, more manageable chunks, making it easier to display data in a more user-friendly way. However, implementing pagination can be a tedious and repetitive task.

That's where `paginate-prisma` comes in. `paginate-prisma` is a TypeScript library that wraps pagination for Prisma ORM. It provides a simple API for pagination that makes it easy to add pagination functionality to your Prisma queries.

In this article, we will explore `paginate-prisma` and learn how to use it.

## Installing `paginate-prisma`

To install `paginate-prisma`, you can use your favorite package manager. Here, we'll use `npm`:

```bash
npm install paginate-prisma
```

## Using `paginate-prisma`

To use `paginate-prisma`, you'll need to import it into your project:

```typescript
import { paginate, PAGINATION_ORDER } from 'paginate-prisma';
```

### Pagination Options

Before we start using `paginate-prisma`, let's take a look at the pagination options available:

| Option | Type | Description |
| --- | --- | --- |
| `page` | `number` | The current page to retrieve. Defaults to `1`. |
| `limit` | `number` | The number of items to retrieve per page. Defaults to `10`. |
| `sort.field` | `string` | The field to sort by. Required if `sort` is specified. |
| `sort.order` | `PAGINATION_ORDER` enum | The sort order (`ASC` or `DESC`). Required if `sort` is specified. |
| `disablePagination` | `boolean` | If true, `paginate-prisma` will not apply pagination and will return all items. Useful for testing or when you don't need paging. |

### Using `paginate`

Once you have imported `paginate-prisma`, you can start using it in your code. To use `paginate`, you need to call it with the following parameters:

```typescript
await paginate(prismaModel)(query, sortingOptions, additionalFindMany);
```

`paginate` returns a Promise that resolves to a `PaginationResult` object and an array of paginated data.

The first parameter is the Prisma model to use for pagination. You'll need to pass the model, for example:

```typescript
const prisma = new PrismaClient();

const data = await paginate(prisma.user)({ /* your query options */ });
```

In the second parameter, you'll specify the pagination options. Here's an example:

```typescript
{
  page: 1,
  limit: 10,
  sort: {
    field: 'createdAt',
    order: PAGINATION_ORDER.ASC,
  },
}
```

This specifies that we want to retrieve the first page of results, with a limit of 10 items per page, sorted by the `createdAt` field in ascending order.

The third parameter is optional and allows you to specify additional parameters to the `findMany` function. For example, you might use this parameter to include related records.

Here's an example that demonstrates how to use `paginate`:

```typescript
import { paginate, PAGINATION_ORDER } from 'paginate-prisma';

const prisma = new PrismaClient();

const data = await paginate(prisma.user)({}, {
  page: 1,
  limit: 10
});
```

## Graphql Types

When using `paginate-prisma` with GraphQL, it can be helpful to define custom GraphQL input and output types that match the types used by the plugin. Here's an example of how to define these types in TypeScript with `type-graphql`.

```typescript
import { Max, Min } from 'class-validator';
import {
  PaginationOptions as POptions,
  PaginationOrder as POrder,
  PaginationResult as PResult,
} from 'paginate-prisma';
import {
  Field,
  InputType,
  Int,
  InterfaceType,
  registerEnumType,
} from 'type-graphql';

enum PAGINATION_ORDER {
  ASC = 'ASC',
  DESC = 'DESC',
}
registerEnumType(PAGINATION_ORDER, {
  name: 'PAGINATION_ORDER',
  description: undefined,
});

@InputType()
export class PaginationOrder<T> implements POrder<T> {
  @Field(() => String)
  field!: T;

  @Field(() => PAGINATION_ORDER, { defaultValue: PAGINATION_ORDER.ASC })
  order!: PAGINATION_ORDER;
}

@InputType()
export class PaginationOptions<T> implements POptions<T> {
  @Min(1)
  @Field(() => Int, { defaultValue: 1, nullable: true })
  page?: number;

  @Field(() => Boolean, { defaultValue: false, nullable: true })
  disablePagination?: boolean;

  @Min(1)
  @Max(20)
  @Field(() => Int, { defaultValue: 10, nullable: true })
  limit?: number;

  @Field(() => PaginationOrder, { nullable: true })
  sort?: PaginationOrder<T>;
}

@InterfaceType()
export class PaginationResult implements PResult {
  @Field(() => Int)
  page!: number;

  @Field(() => Int)
  pages!: number;

  @Field(() => Int)
  limit!: number;

  @Field(() => Int)
  items!: number;
}
```

## Conclusion

In conclusion, `paginate-prisma` is a simple and efficient package for handling pagination in your Prisma ORM applications. With its easy-to-use API and extensive documentation, it can help developers focus on building their applications instead of worrying about pagination logic. Its integration with TypeScript and TypeGraphQL also makes it a great option for building type-safe applications.

By using `paginate-prisma`, you can reduce the amount of boilerplate code needed to handle pagination and simplify your codebase. The package is actively maintained, with regular updates and bug fixes being released to ensure compatibility with the latest versions of Prisma and other dependencies.

Overall, if you're building an application with Prisma ORM and need to handle pagination, `paginate-prisma` is definitely a package worth considering.
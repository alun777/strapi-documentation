---
title: Filtering for REST API - Strapi Developer Docs
description: Use Strapi's REST API to filter the results of your requests.
sidebarDepth: 3
canonicalUrl: https://docs.strapi.io/developer-docs/latest/developer-resources/database-apis-reference/rest-api/filtering-locale-publication.html
---

# REST API: Filtering, Locale, and Publication State

The [REST API](/developer-docs/latest/developer-resources/database-apis-reference/rest-api.md) offers the ability to filter results found with its ["Get entries"](/developer-docs/latest/developer-resources/database-apis-reference/rest-api.md#get-entries) method.<br/>
Using optional Strapi features can provide some more filters:

- If the [Internationalization (i18n) plugin](/developer-docs/latest/plugins/i18n.md) is enabled on a content-type, it's possible to filter by locale.
- If the [Draft & Publish](/developer-docs/latest/concepts/draft-and-publish.md) is enabled, it's possible to filter based on a `live` or `draft` state.

## Filtering

Queries can accept a `filters` parameter with the following syntax:

`GET /api/:pluralApiId?filters[field][operator]=value`

The following operators are available:

| Operator        | Description                              |
| --------------- | ---------------------------------------- |
| `$eq`           | Equal                                    |
| `$ne`           | Not equal                                |
| `$lt`           | Less than                                |
| `$lte`          | Less than or equal to                    |
| `$gt`           | Greater than                             |
| `$gte`          | Greater than or equal to                 |
| `$in`           | Included in an array                     |
| `$notIn`        | Not included in an array                 |
| `$contains`     | Contains (case-sensitive)                |
| `$notContains`  | Does not contain (case-sensitive)        |
| `$containsi`    | Contains                                 |
| `$notContainsi` | Does not contain                         |
| `$null`         | Is null                                  |
| `$notNull`      | Is not null                              |
| `$between`      | Is between                               |
| `$startsWith`   | Starts with                              |
| `$endsWith`     | Ends with                                |
| `$or`           | Joins the filters in an "or" expression  |
| `$and`          | Joins the filters in an "and" expression |

### Find users having 'John' as first name

::::api-call
:::request Example request

```js
const qs = require('qs');
const query = qs.stringify({
  filters: {
    username: {
      $eq: 'John',
    },
  },
}, {
  encodeValuesOnly: true,
});

await request(`/api/users?${query}`);
// GET /api/users?filters[username][$eq]=John
```

:::

:::response Example response

```json
[
  {
    "id": 1,
    "username": "John",
    "email": "john@test.com",
    "provider": "local",
    "confirmed": true,
    "blocked": false,
    "createdAt": "2021-12-03T20:08:17.740Z",
    "updatedAt": "2021-12-03T20:08:17.740Z"
  }
]
```

:::
::::

### Find multiple restaurants with ids 3, 6, 8

::::api-call
:::request Example request

```js
const qs = require('qs');
const query = qs.stringify({
  filters: {
    id: {
      $in: [3, 6, 8],
    },
  },
}, {
  encodeValuesOnly: true, // prettify url
});

await request(`/api/restaurants?${query}`);
// GET /api/restaurants?filters[id][$in][0]=3&filters[id][$in][1]=6&filters[id][$in][2]=8
```

:::

:::response Example response

```json
{
  "data": [
    {
      "id": 3,
      "attributes": {
        "name": "test3",
        // ...
      }
    },
    {
      "id": 6,
      "attributes": {
        "name": "test6",
        // ...
      }
    },
    {
      "id": 8,
      "attributes": {
        "name": "test8",
        // ...
      }
    }
  ],
  "meta": {
    // ...
  }
}
```

:::
::::

:::caution
By default, the filters can only be used from `find` endpoints generated by the Content-Type Builder and the CLI.
:::

### Complex filtering

Complex filtering is combining multiple filters utilizing advanced methods such as combining `$and` & `$or`. This allows for more flexibility to request exactly the data needed.

::::api-call
:::request Example request: Find books with 2 possible dates & a specific author

```js
const qs = require('qs');
const query = qs.stringify({
  filters: {
    $or: [
      {
        date: {
          $eq: '2020-01-01',
        },
      },
      {
        date: {
          $eq: '2020-01-02',
        },
      },
    ],
    author: {
      name: {
        $eq: 'Kai doe',
      },
    },
  },
}, {
  encodeValuesOnly: true,
});

await request(`/api/books?${query}`);
// GET /api/books?filters[$or][0][date][$eq]=2020-01-01&filters[$or][1][date][$eq]=2020-01-02&filters[author][name][$eq]=Kai%20doe
```

:::

:::response Example response

```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "name": "test1",
        "date": "2020-01-01",
        // ...
      }
    },
    {
      "id": 2,
      "attributes": {
        "name": "test2",
        "date": "2020-01-02",
        // ...
      }
    }
  ],
  "meta": {
    // ...
  }
}
```

:::
::::

### Deep filtering

Deep filtering is filtering on a relation's fields.

::::api-call
:::request Example request: Find restaurants owned by a chef who belongs to a 5-star restaurant

```js
const qs = require('qs');
const query = qs.stringify({
  filters: {
    chef: {
      restaurants: {
        stars: {
          $eq: 5,
        },
      },
    },
  },
}, {
  encodeValuesOnly: true,
});

await request(`/api/restaurants?${query}`);
// GET /api/restaurants?filters[chef][restaurants][stars][$eq]=5
```

:::

:::response Example response

```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "name": "GORDON RAMSAY STEAK",
        "stars": 5
        // ...
      }
    },
    {
      "id": 2,
      "attributes": {
        "name": "GORDON RAMSAY BURGER",
        "stars": 5
        // ...
      }
    }
  ],
  "meta": {
    // ...
  }
}
```

:::
::::

::: caution

- Querying your API with deep filters may cause performance issues.  If one of your deep filtering queries is too slow, we recommend building a custom route with an optimized version of the query.
- Deep filtering isn't available for polymorphic relations (eg: Dynamic Zones & Media Fields).

:::

:::note

- Relations, media fields, components, and dynamic zones are not populated by default. Use the `populate` parameter to populate these data structures (see [population documentation](/developer-docs/latest/developer-resources/database-apis-reference/rest/populating-fields.md#population))
- It is not possible to filter on dynamic zones or media fields

:::

## Locale

:::prerequisites

- The [Internationalization (i18n) plugin](/developer-docs/latest/plugins/i18n.md) should be installed.
- [Localization should be enabled for the content-type](/user-docs/latest/content-types-builder/creating-new-content-type.md#creating-a-new-content-type).
:::

The `locale` API parameter can be used to [get entries from a specific locale](/developer-docs/latest/plugins/i18n.md#getting-localized-entries-with-the-locale-parameter).

## Publication State

:::prerequisites
The [Draft & Publish](/developer-docs/latest/concepts/draft-and-publish.md) feature should be enabled.
:::

Queries can accept a `publicationState` parameter to fetch entries based on their publication state:

- `live`: returns only published entries (default)
- `preview`: returns both draft entries & published entries

::::api-call
:::request Example request: Get both published and draft articles

```js
const qs = require('qs');
const query = qs.stringify({
  publicationState: 'preview',
}, {
  encodeValuesOnly: true,
});

await request(`/api/articles?${query}`);
// GET /api/articles?publicationState=preview
```

:::
:::response Example response

```json
{
  "data": [
    {
      "id": 1,
      "attributes": {
        "title": "This a Draft",
        "publishedAt": null
        // ...
      }
    },
    {
      "id": 2,
      "attributes": {
        "title": "This is Live",
        "publishedAt": "2021-12-03T20:08:17.740Z"
        // ...
      }
    }
  ],
  "meta": {
    // ...
  }
}
```

:::
::::

::::tip
To retrieve only draft entries, combine the `preview` publication state and the `publishedAt` fields:

`GET /api/articles?publicationState=preview&filters[publishedAt][$null]=true`

:::details Example using qs

```js
const qs = require('qs');
const query = qs.stringify({
  publicationState: 'preview',
  filters: {
    publishedAt: {
      $null: true,
    },
  },
}, {
  encodeValuesOnly: true,
});

await request(`/api/articles?${query}`);
// GET /api/articles?publicationState=preview&filters[publishedAt][$null]=true
```

:::
::::

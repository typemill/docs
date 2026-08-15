#  get items

Returns one or more items for articles with a slug.

## Usage

This is the preferred way if articles are often moved to other folders. The relative URL will change each time, but the slug will stay the same. The endpoint will return an array of items because slugs are not unique, as a website can contain several pages with the same slug in different folders. If a slug itself is changed, the article will no longer be found.

## Authorization

The minimum role for authorization is "author".

## Endpoint

```http
GET /api/v1/article/items
```

## Query parameters

| Query parameter | Type   | Required? | Description   |
|:----------------|:-------|:----------|:--------------|
| slug            | string | required  | the slug       |

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/article/items?slug=published'
```

## Response Example

```json
{
  "items": [
    {
      "originalName": "04-published.md",
      "elementType": "file",
      "status": "published",
      "fileType": "md",
      "order": "04",
      "name": "published",
      "slug": "published",
      "path": "/00-getting-started/04-published.md",
      "pathWithoutType": "/00-getting-started/04-published",
      "key": 4,
      "keyPath": "0.4",
      "keyPathArray": [
        "0",
        "4"
      ],
      "chapter": "1.5",
      "urlRelWoF": "/getting-started/published",
      "urlRel": "/typemill/getting-started/published",
      "urlAbs": "http://localhost/typemill/getting-started/published",
      "active": false,
      "activeParent": false,
      "hide": false,
      "noindex": false
    },
    {
      "originalName": "02-published.md",
      "elementType": "file",
      "status": "published",
      "fileType": "md",
      "order": "02",
      "name": "published",
      "slug": "published",
      "path": "/01-publish-status/02-published.md",
      "pathWithoutType": "/01-publish-status/02-published",
      "key": 2,
      "keyPath": "1.2",
      "keyPathArray": [
        "1",
        "2"
      ],
      "chapter": "2.3",
      "urlRelWoF": "/publish-status/published",
      "urlRel": "/typemill/publish-status/published",
      "urlAbs": "http://localhost/typemill/publish-status/published",
      "active": false,
      "activeParent": false,
      "hide": false,
      "noindex": false
    }
  ]
}
```


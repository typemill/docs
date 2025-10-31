# get meta

Returns the meta data of an article that are stored in the meta-tabs.

## Authorization

The minimum role for authorization is "author".

## Endpoint

```http
GET /api/v1/article/meta
```

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| url | string | required | relative url |

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/article/meta?url=/getting-started'
```

## Response Example

```json
{
    "metadata": {
        "meta": {
            "navtitle": "getting started",
            "title": "Getting Started with Typemill",
            "description": "Use this demo-content to familiarize yourself with Typemill. Not sure where to start?",
            "heroimage": null,
            "heroimagealt": null,
            "owner": "trendschau",
            "author": "Sebastian Schürmanns",
            "allowedrole": null,
            "alloweduser": null,
            "manualdate": null,
            "modified": "2024-05-17",
            "created": "2024-04-25",
            "time": "13-16-58",
            "reference": null,
            "referencetype": null,
            "hide": false,
            "noindex": false,
            "contains": "pages",
            "glossary": null,
            "template": "",
            "pagelisting": null,
            "shorttitle": null
        },
        "demo": {
            "demoimage": null,
            "demoimagealt": null,
            "democheckbox": null,
            "democustomfield": null
        }
    },
}
```


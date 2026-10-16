# get meta

Returns the meta data of an article that are stored in the meta-tabs.

## Authorization

All authenticated user roles are authorized for published pages. Unpublished pages are only visible to their owner and to users with the role "author" or higher; all other users get a `404` response. If a page is restricted to a user role or a list of users in the page metadata, all other users get a `403` response.

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

The response contains a `meta` object with one key per meta-tab. The standard tab is also called `meta`, additional tabs can be added by plugins:

```json
{
    "meta": {
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
    }
}
```

## Related endpoint for authors

The author interface uses another endpoint that returns the metadata together with the meta-definitions (the form field definitions of all tabs):

```http
GET /api/v1/meta?url=/getting-started
```

This endpoint requires the role "contributor" or higher and returns the keys `metadata` and `metadefinitions`. Use `GET /api/v1/article/meta` if you only need the metadata values of a page.

# get article

Returns the content of an article as an array of blocks with the id, markdown and html of each block.

## Usage

You can get the article content directly with the relative url. If the relative url changes often, because the article is moved to another folder, then it makes sense to search for the slug first (get items) and get the relative url dynamically.

## Authorization

The minimum role for authorization is "author".

## Endpoint

```http
GET /api/v1/article/content
```

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| url | string | required | relative url | 
| draft | bool | no | return draft version if exists |

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/article/content?url=/getting-started&draft=true'
```

## Response Example

```json
{
    "content": [
        {
            "id": 0,
            "markdown": "# Draft: Getting Started with Typemill",
            "html": "<h1 id=\"h-draft-getting-started-with-typemill\">Draft: Getting Started with Typemill<\/h1>"
        },
        {
            "id": 1,
            "markdown": "Use this demo-content to familiarize yourself with Typemill.",
            "html": "<p>Use this demo-content to familiarize yourself with Typemill.<\/p>"
        },
        {
            "id": 2,
            "markdown": "Not sure where to start?",
            "html": "<p>Not sure where to start?<\/p>"
        },
        {
            "id": 3,
            "markdown": "Simply select a topic in the navigation; it will briefly explain what you can do with Typemill.",
            "html": "<p>Simply select a topic in the navigation; it will briefly explain what you can do with Typemill.<\/p>"
        }
    ],
}
```


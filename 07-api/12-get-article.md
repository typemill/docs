# get article

Returns the content of an article, either as an array of blocks with the id, markdown and html of each block, or as a raw markdown document.

## Usage

You can get the article content directly with the relative url. If the relative url changes often, because the article is moved to another folder, then it makes sense to search for the slug first (get items) and get the relative url dynamically.

Use the query parameter `format=raw` if you want to work with the original markdown of the page, for example for external integrations or AI tools. The default block format is primarily useful for the author interface.

## Authorization

All authenticated user roles are authorized for published pages. Unpublished pages are only visible to their owner and to users with the role "author" or higher; all other users get a `404` response. If a page is restricted to a user role or a list of users in the page metadata, all other users get a `403` response.

The query parameter `draft=true` requires the role "contributor" or higher, and users below the role "author" can only access drafts of their own pages.

## Endpoint

```http
GET /api/v1/article/content
```

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| url | string | required | relative url | 
| draft | bool | no | return draft version if exists |
| format | string | no | `raw` returns the content as one raw markdown string instead of a block array. |

Note that a page that has never been published returns a `404` response, unless you request it with `draft=true` and have the required rights.

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/article/content?url=/getting-started&draft=true'
```

## Response Example (default)

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
    ]
}
```

## Request Example (raw markdown)

```curl
curl -X GET 'http://localhost/typemill/api/v1/article/content?url=/getting-started&format=raw'
```

## Response Example (raw markdown)

For published pages the raw format returns the original markdown file unchanged:

```json
{
    "content": "# Getting Started with Typemill\n\nUse this demo-content to familiarize yourself with Typemill.\n\nNot sure where to start?\n\nSimply select a topic in the navigation; it will briefly explain what you can do with Typemill."
}
```

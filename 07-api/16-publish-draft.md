# publish draft

Saves the content of a page and publishes it directly. This is a combination of [update draft](/api/update-draft) and [publish article](/api/publish-article) in a single request.

## Usage

Accepts the same parameters as [update draft](/api/update-draft): either the full markdown document with `markdown` or the `title` and `body` split.

If you want to publish an existing draft without changing the content, use [publish article](/api/publish-article) instead.

## Authorization

The minimum role for authorization is "author". Users with the role "author" can only publish pages they own (listed in the `owner` field of the page metadata), users with the role "editor" or higher can publish all pages.

## Endpoint

```http
POST /api/v1/draft/publish
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| url | string | required | The relative url of the page. | 
| markdown | string | conditional | The full markdown document of the page. Required if no `title` and `body` are given. | 
| title | string | conditional | The first markdown block of the page including the markdown prefix, e.g. `# My title`. Required together with `body` if no `markdown` is given. | 
| body | string | conditional | The remaining markdown of the page. Required together with `title` if no `markdown` is given. | 
| item_id | string | optional | The internal keypath of the page (e.g. `0.4`). Not used anymore and only validated for backwards compatibility. | 

## Request Example

```curl
curl -X POST 'http://localhost/typemill/api/v1/draft/publish' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"url": "/getting-started/my-new-page", "markdown": "# My new page\n\nThis is the new content."}'
```

## Response Example

The response contains the published item, the refreshed draft navigation and the content as a block array with id, markdown and html:

```json
{
    "item": {
        "originalName": "04-my-new-page.md",
        "elementType": "file",
        "status": "published",
        "slug": "my-new-page",
        "urlRelWoF": "/getting-started/my-new-page",
        "keyPath": "0.4"
    },
    "navigation": [],
    "content": [
        {
            "id": 0,
            "markdown": "# My new page",
            "html": "<h1 id=\"my-new-page\">My new page<\/h1>"
        },
        {
            "id": 1,
            "markdown": "This is the new content.",
            "html": "<p>This is the new content.<\/p>"
        }
    ]
}
```

The response example is shortened.

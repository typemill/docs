# update draft

Saves the content of a page as a draft. The live version of the page stays unchanged until you publish the draft.

## Usage

There are two ways to send the content:

* With `markdown`, the full markdown document of the page. This is the recommended way for external integrations.
* With `title` and `body`, the format that the author interface uses. The `title` is the first markdown block of the page including the markdown prefix (e.g. `# My title`), the `body` is the remaining markdown.

## Authorization

The minimum role for authorization is "author". Users with the role "author" can only update pages they own (listed in the `owner` field of the page metadata), users with the role "editor" or higher can update all pages.

## Endpoint

```http
PUT /api/v1/draft
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
curl -X PUT 'http://localhost/typemill/api/v1/draft' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"url": "/getting-started/my-new-page", "markdown": "# My new page\n\nThis is the new content."}'
```

## Response Example

The response contains the updated item, the refreshed draft navigation and the draft content as a block array with id, markdown and html:

```json
{
    "item": {
        "originalName": "04-my-new-page.txt",
        "elementType": "file",
        "status": "unpublished",
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

The response example is shortened. The status of the item changes to `modified` once you update the draft of a published page.

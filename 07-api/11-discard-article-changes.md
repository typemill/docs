# Discard Article Changes

Discards the draft changes of a page. The last published version of the page stays live and unchanged.

## Usage

Use this endpoint to delete the draft of a page without unpublishing or deleting the page itself.

## Authorization

The minimum role for authorization is "editor".

## Endpoint

```http
DELETE /api/v1/article/discard
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| item_id | string | required | The internal keypath of the page (e.g. `0.4`). You can find keypaths in the response of [get navigation](/api/get-navigation). | 
| url | string | required | The relative url of the page. | 

## Request Example

```curl
curl -X DELETE 'http://localhost/typemill/api/v1/article/discard' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"item_id": "0.4", "url": "/getting-started/my-new-page"}'
```

## Response Example

The response contains the item, the refreshed draft navigation and the content of the published version as a block array with id, markdown and html:

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
        }
    ]
}
```

The response example is shortened.

## Error responses

| Status code | Description | 
|---|---|
| 400 | Missing or invalid parameters, e.g. no `item_id`. The response contains an `errors` object with details. | 
| 403 | The page is outside the folders the user is restricted to (folder access). | 
| 404 | The page was not found. | 


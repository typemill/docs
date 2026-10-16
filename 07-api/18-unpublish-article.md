# unpublish article

Unpublishes a page. The live version is removed from the website, the content is kept as a draft.

## Usage

If you unpublish a folder that contains published pages, these pages are not visible on the website anymore. In this case the response contains a warning message.

## Authorization

The minimum role for authorization is "editor".

## Endpoint

```http
DELETE /api/v1/article/unpublish
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| item_id | string | required | The internal keypath of the page (e.g. `0.4`). You can find keypaths in the response of [get navigation](/api/get-navigation). | 
| url | string | required | The relative url of the page. | 

## Request Example

```curl
curl -X DELETE 'http://localhost/typemill/api/v1/article/unpublish' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"item_id": "0.4", "url": "/getting-started/my-new-page"}'
```

## Response Example

The response contains the refreshed draft navigation and the unpublished item:

```json
{
    "message": false,
    "navigation": [],
    "item": {
        "originalName": "04-my-new-page.txt",
        "elementType": "file",
        "status": "unpublished",
        "slug": "my-new-page",
        "urlRelWoF": "/getting-started/my-new-page",
        "keyPath": "0.4"
    }
}
```

The response example is shortened.

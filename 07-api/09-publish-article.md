# Publish Article

Publishes the existing draft of a page. The content of the page is not changed with this call.

## Usage

Use this endpoint to publish a draft that you have created or updated before with [update draft](/api/update-draft). If you want to save new content and publish it in a single request, use [publish draft](/api/publish-draft) instead.

## Authorization

The minimum role for authorization is "editor".

## Endpoint

```http
POST /api/v1/article/publish
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| item_id | string | required | The internal keypath of the page (e.g. `0.4`). Can be an empty string for the startpage. You can find keypaths in the response of [get navigation](/api/get-navigation). | 
| url | string | required | The relative url of the page. | 

## Request Example

```curl
curl -X POST 'http://localhost/typemill/api/v1/article/publish' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"item_id": "0.4", "url": "/getting-started/my-new-page"}'
```

## Response Example

The response contains the refreshed draft navigation, the published item and its metadata:

```json
{
    "navigation": [],
    "item": {
        "originalName": "04-my-new-page.md",
        "elementType": "file",
        "status": "published",
        "slug": "my-new-page",
        "urlRelWoF": "/getting-started/my-new-page",
        "keyPath": "0.4"
    },
    "metadata": {
        "meta": {
            "navtitle": "my new page",
            "title": "My new page"
        }
    },
    "message": ""
}
```

The response example is shortened.


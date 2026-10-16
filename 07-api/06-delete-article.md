# Delete Article

Deletes a page or a folder with all its content. This removes the published version, the draft and the metadata of the page.

## Usage

If you delete a folder, all pages within the folder are deleted, too. This action cannot be undone, so use it with care.

## Authorization

The minimum role for authorization is "editor".

## Endpoint

```http
DELETE /api/v1/article
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| item_id | string | required | The internal keypath of the page (e.g. `0.4`). You can find keypaths in the response of [get navigation](/api/get-navigation). | 
| url | string | required | The relative url of the page. | 

## Request Example

```curl
curl -X DELETE 'http://localhost/typemill/api/v1/article' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"item_id": "0.4", "url": "/getting-started/my-new-page"}'
```

## Response Example

The response contains a redirect url to the parent item, which is used by the author interface:

```json
{
    "url": "http://localhost/typemill/tm/content/visual/getting-started"
}
```


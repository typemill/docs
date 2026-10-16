# create article

Creates a new page or folder.

## Usage

The new page is created as an **unpublished draft** with a placeholder content (`# {item_name}` followed by `Content`). Use [update draft](/api/update-draft) to add your own markdown content and one of the publish endpoints to make the page visible.

You can define the parent folder in two ways:

* With `parent_url`, the relative url of the parent folder. This is the recommended way for external integrations.
* With `folder_id`, the internal keypath of the parent folder (e.g. `0.1`) or `root` for the base level. You can find keypaths in the response of [get navigation](/api/get-navigation).

## Authorization

The minimum role for authorization is "author".

## Endpoint

```http
POST /api/v1/article
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| item_name | string | required | The name of the new page or folder (1-60 characters). The name is transformed into a slug for the url. | 
| type | string | required | `file` for a page or `folder` for a folder. | 
| parent_url | string | optional | The relative url of the parent folder, e.g. `/getting-started`. Use `/` for the root level. Required if no `folder_id` is given. | 
| folder_id | string | optional | The internal keypath of the parent folder (e.g. `0.1`) or `root` for the base level. Required if no `parent_url` is given. | 
| url | string | optional | A relative url used to detect the project or language in multilanguage or multiproject setups. Falls back to `parent_url`. | 

## Request Example

```curl
curl -X POST 'http://localhost/typemill/api/v1/article' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"item_name": "My new page", "type": "file", "parent_url": "/getting-started"}'
```

## Response Example

The response contains the refreshed draft navigation and the created `item` with its relative url (`urlRelWoF`), its keypath and its status:

```json
{
    "navigation": [
        {
            "originalName": "00-getting-started",
            "elementType": "folder",
            "slug": "getting-started"
        }
    ],
    "item": {
        "originalName": "04-my-new-page.txt",
        "elementType": "file",
        "status": "unpublished",
        "fileType": "txt",
        "name": "my new page",
        "slug": "my-new-page",
        "path": "/00-getting-started/04-my-new-page.txt",
        "urlRelWoF": "/getting-started/my-new-page",
        "urlRel": "/typemill/getting-started/my-new-page",
        "urlAbs": "http://localhost/typemill/getting-started/my-new-page",
        "key": 4,
        "keyPath": "0.4",
        "keyPathArray": [
            "0",
            "4"
        ],
        "chapter": "1.5"
    },
    "message": "",
    "url": false
}
```

The response examples on this page are shortened. The `url` field is always `false` and only kept for backwards compatibility with the author interface.

## Error responses

| Status code | Description | 
|---|---|
| 400 | Missing or invalid parameters. | 
| 402 | A page with this name already exists in the folder, or the name is reserved (e.g. `tm` on root level). | 
| 404 | The parent folder was not found or is not a folder. | 

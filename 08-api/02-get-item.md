# get item

Returns an item for an article with the relative url.

## Usage

This is the prefered way to get an item if the relative url will not change. If a page is moved to another folder in the interface, then the relative url will change and the item will not be found. In this case use [get items](/api/getitems) to search for a slug instead of a relative url.

## Authorization

The minimum role for authorization is "author".

## Endpoint

```http
GET /api/v1/article/item
```

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| url | string | required | relative url | 

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/article/item?url=/getting-started'
```

## Response Example

```json

{
    "item": {
        "originalName": "00-getting-started",
        "elementType": "folder",
        "contains": "pages",
        "status": "modified",
        "fileType": "txt",
        "order": "00",
        "name": "getting started",
        "slug": "getting-started",
        "path": "/00-getting-started",
        "pathWithoutType": "/00-getting-started/index",
        "urlRelWoF": "/getting-started",
        "urlRel": "/typemill/getting-started",
        "urlAbs": "http://localhost/typemill/getting-started",
        "key": 0,
        "keyPath": 0,
        "keyPathArray": [
            "0"
        ],
        "chapter": 1,
        "active": false,
        "activeParent": false,
        "hide": false,
        "folderContent": [
            {
                "originalName": "00-create-your-first-page.md",
                "elementType": "file",
                "status": "published",
                "fileType": "md",
                "order": "00",
                "name": "create your first page",
                "slug": "create-your-first-page",
                "path": "/00-getting-started/00-create-your-first-page.md",
                "pathWithoutType": "/00-getting-started/00-create-your-first-page",
                "key": 0,
                "keyPath": "0.0",
                "keyPathArray": [
                    "0",
                    "0"
                ],
                "chapter": "1.1",
                "urlRelWoF": "/getting-started/create-your-first-page",
                "urlRel": "/typemill/getting-started/create-your-first-page",
                "urlAbs": "http://localhost/typemill/getting-started/create-your-first-page",
                "active": false,
                "activeParent": false,
                "hide": false,
                "noindex": false
            },
            {
                "originalName": "01-edit-your-page.md",
                "elementType": "file",
                "status": "published",
                "fileType": "md",
                "order": "01",
                "name": "edit your page",
                "slug": "edit-your-page",
                "path": "/00-getting-started/01-edit-your-page.md",
                "pathWithoutType": "/00-getting-started/01-edit-your-page",
                "key": 1,
                "keyPath": "0.1",
                "keyPathArray": [
                    "0",
                    "1"
                ],
                "chapter": "1.2",
                "urlRelWoF": "/getting-started/edit-your-page",
                "urlRel": "/typemill/getting-started/edit-your-page",
                "urlAbs": "http://localhost/typemill/getting-started/edit-your-page",
                "active": false,
                "activeParent": false,
                "hide": false,
                "noindex": false
            },
            {
                "originalName": "02-edit-the-page-meta.md",
                "elementType": "file",
                "status": "published",
                "fileType": "md",
                "order": "02",
                "name": "edit the page meta",
                "slug": "edit-the-page-meta",
                "path": "/00-getting-started/02-edit-the-page-meta.md",
                "pathWithoutType": "/00-getting-started/02-edit-the-page-meta",
                "key": 2,
                "keyPath": "0.2",
                "keyPathArray": [
                    "0",
                    "2"
                ],
                "chapter": "1.3",
                "urlRelWoF": "/getting-started/edit-the-page-meta",
                "urlRel": "/typemill/getting-started/edit-the-page-meta",
                "urlAbs": "http://localhost/typemill/getting-started/edit-the-page-meta",
                "active": false,
                "activeParent": false,
                "hide": false,
                "noindex": false
            },
            {
                "originalName": "03-publish-your-page.md",
                "elementType": "file",
                "status": "published",
                "fileType": "md",
                "order": "03",
                "name": "publish your page",
                "slug": "publish-your-page",
                "path": "/00-getting-started/03-publish-your-page.md",
                "pathWithoutType": "/00-getting-started/03-publish-your-page",
                "key": 3,
                "keyPath": "0.3",
                "keyPathArray": [
                    "0",
                    "3"
                ],
                "chapter": "1.4",
                "urlRelWoF": "/getting-started/publish-your-page",
                "urlRel": "/typemill/getting-started/publish-your-page",
                "urlAbs": "http://localhost/typemill/getting-started/publish-your-page",
                "active": false,
                "activeParent": false,
                "hide": false,
                "noindex": false
            },
            {
                "originalName": "04-template.txt",
                "elementType": "file",
                "status": "unpublished",
                "fileType": "txt",
                "order": "04",
                "name": "template",
                "slug": "template",
                "path": "/00-getting-started/04-template.txt",
                "pathWithoutType": "/00-getting-started/04-template",
                "key": 4,
                "keyPath": "0.4",
                "keyPathArray": [
                    "0",
                    "4"
                ],
                "chapter": "1.5",
                "urlRelWoF": "/getting-started/template",
                "urlRel": "/typemill/getting-started/template",
                "urlAbs": "http://localhost/typemill/getting-started/template",
                "active": false,
                "activeParent": false,
                "hide": false,
                "noindex": false
            }
        ],
        "noindex": false
    }
  }
}
```


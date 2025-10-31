# get navigation

Returns an whole content navigation with all published pages or with all published and unpublished pages.

## Usage

Get the whole content navigation as a multi-dimensional array of item objects. Use this to have full access to the whole content.

## Authorization

The minimum role for authorization is "member".

## Endpoint

```http
GET /api/v1/navigation
```

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| draft | bool | optional | `true` if you want to get the navigation with published and unpublished pages. | 

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/navigation?draft=true'
```

## Response Example

```json

{
  "navigation": [
    {
      "originalName": "00-getting-started",
      "elementType": "folder",
      "contains": "pages",
      "status": "published",
      "fileType": "md",
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
        ...
    }
  ]
}

```


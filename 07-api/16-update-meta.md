# Update Meta

Updates the metadata of a page. The metadata is organized in tabs, and each tab holds a set of fields. The standard tab is called `meta`, plugins can add more tabs.

## Usage

You can only update one tab per request. Send the field values of the tab as an object in the `data` parameter. You can read the current values with [get meta](/api/get-meta).

To find out which tabs and fields exist in your installation, use the author endpoint `GET /api/v1/meta?url={url}`, which returns the metadata together with the meta-definitions (the form field definitions of all tabs).

## Authorization

The minimum role for authorization is "contributor". Users with the role "contributor" and "author" can only update the metadata of pages they own (listed in the `owner` field of the page metadata), users with the role "editor" or higher can update all pages.

## Endpoint

```http
POST /api/v1/meta
```

## Body parameters

| Body parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| url | string | required | The relative url of the page. | 
| tab | string | required | The name of the meta-tab, e.g. `meta`. | 
| data | object | required | An object with the field names and values of the tab. | 

## Request Example

```curl
curl -X POST 'http://localhost/typemill/api/v1/meta' \
     -u username:password \
     -H 'Content-Type: application/json' \
     -d '{"url": "/getting-started/my-new-page", "tab": "meta", "data": {"title": "My new page", "description": "A page created with the api."}}'
```

## Response Example

The response contains the refreshed draft navigation and the updated item. If a manual date change renamed a post, the `rename` field contains the result:

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
    "rename": false
}
```

The response example is shortened.

## Error responses

| Status code | Description | 
|---|---|
| 400 | Missing or invalid parameters, or the field values in `data` did not pass the field validation. The response contains an `errors` object with details. | 
| 404 | The page or the meta-tab was not found. | 


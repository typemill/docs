# get settings

Get the system settings. This endpoint is probably not of interest, but you could 

## Authorization

Minimum role for authorization is "manager".

## Endpoint

```http
GET /api/v1/mainnavi/
```

## Request Example

```curl
curl -X GET 'https://yourwebsite.com/api/v1/mainnavi'
```

## Response Example

```json

{
  "settings": {
    "version": "2.14.0",
    "title": "Typemill",
    "author": "Sebastian",
    "copyright": "CC-BY",
    "language": "en",
    "langattr": "de",
    "editor": "visual",
    "storage": "\\Typemill\\Models\\Storage",
    "formats": [
      "table",
      "quote",
      "notice",
      "image",
      "video",
      "file",
      "toc",
      "hr",
      "definition",
      "code",
      "shortcode",
      "markdown",
      "headline",
      "ulist",
      "olist",
      "audio"
    ],
    "images": {
      "live": {
        "width": 820
      },
      "thumbs": {
        "width": 250,
        "height": 150
      }
    },
    "systemSettingsPath": "/var/www/html/typemill/system/typemill/settings/",
    "plugins": {
      "demo": {
        "active": true
      },
      "search": {
        "placeholder": "Text for my text",
        "resulttext": "Result",
        "noresulttext": "bla",
        "active": false
      }
    },
    ...
}

```


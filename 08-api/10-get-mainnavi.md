# get mainnavi

Get the main navigation items. The response will vary according to the permissions of the user role.

## Authorization

The minimum role for authorization is "member".

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
  "mainnavi": {
    "content": {
      "title": "Content",
      "routename": "content.visual",
      "aclresource": "content",
      "aclprivilege": "read"
    },
    "system": {
      "title": "System",
      "routename": "settings.show",
      "aclresource": "system",
      "aclprivilege": "read",
      "active": true
    },
    "frontend": {
      "title": "Frontend",
      "routename": "home",
      "aclresource": "account",
      "aclprivilege": "read"
    },
    "logout": {
      "title": "Logout",
      "routename": "auth.logout",
      "aclresource": "account",
      "aclprivilege": "read"
    }
  }
}
```


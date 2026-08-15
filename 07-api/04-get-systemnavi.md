# get systemnavi

Get the system navigation items. The response will vary depending on the permissions of the user role.

## Authorization

The minimum role for authorization is "member".

## Endpoint

```http
GET /api/v1/systemnavi/
```

## Request Example

```curl
curl -X GET 'https://yourwebsite.com/api/v1/systemnavi'
```

## Response Example

```json
{
  "systemnavi": {
    "system": {
      "title": "System",
      "routename": "settings.show",
      "icon": "icon-wrench",
      "aclresource": "system",
      "aclprivilege": "read",
      "url": "/typemill/tm/system"
    },
    "license": {
      "title": "License",
      "routename": "license.show",
      "icon": "icon-wrench",
      "aclresource": "user",
      "aclprivilege": "read",
      "url": "/typemill/tm/license"
    },
    "themes": {
      "title": "Themes",
      "routename": "themes.show",
      "icon": "icon-paint-brush",
      "aclresource": "system",
      "aclprivilege": "read",
      "url": "/typemill/tm/themes"
    },
    "plugins": {
      "title": "Plugins",
      "routename": "plugins.show",
      "icon": "icon-plug",
      "aclresource": "system",
      "aclprivilege": "read",
      "url": "/typemill/tm/plugins"
    },
    "account": {
      "title": "Account",
      "routename": "user.account",
      "icon": "icon-user",
      "aclresource": "account",
      "aclprivilege": "read",
      "url": "/typemill/tm/account"
    },
    "users": {
      "title": "Users",
      "routename": "users.show",
      "icon": "icon-group",
      "aclresource": "user",
      "aclprivilege": "read",
      "url": "/typemill/tm/users"
    },
    "Demo": {
      "title": "Demo",
      "routename": "demo.admin",
      "icon": "icon-download",
      "aclresource": "system",
      "aclprivilege": "view",
      "url": "/typemill/tm/demo"
    }
  }
}
```


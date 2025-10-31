# get users by role

Get a list of users by userrole.

## Endpoint

```http
GET /api/v1/users/getbyrole
```

## Authorization

The minimum role for authorization is "administrator".

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| role | string | required | Name of the userrole |

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/users/getbyrole?role=administrator'
```

## Response Example

```json
{
  "userdata": [
    {
      "username": "trendschau",
      "email": "test@mymail.com",
      "userrole": "administrator",
      "lastlogin": 1738326436,
      "firstname": "",
      "lastname": "",
      "description": "It is good! Baby!!",
      "image": "",
      "apiaccess": true,
      "darkmode": false,
      "authcodedata": "312312412415152634747",
      "fingerprints": [
        "0f5113d4a2b4751f15a2ed787145978e",
        "6968aacfd1b6ccd3f9faa5db8260ebc5"
      ]
    }
  ]
}
```

5est


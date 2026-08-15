# get user by email

Get a single user by email

## Endpoint

```http
GET /api/v1/users/getbyemail
```

## Authorization

The minimum role for authorization is "administrator".

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| email | string | required | valid email |

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/users/getbyemail?email=test@gmail.com
'
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
      "authcodedata": "asdfs112312314asdfe",
      "fingerprints": [
        "0f5113d4a2b4751f15a2ed787145978e",
        "6968aacfd1b6ccd3f9faa5db8260ebc5"
      ]
    }
  ]
}
```


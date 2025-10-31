# get users by name

Get a list of users by usernames.

## Endpoint

```http
GET /api/v1/users/getbynames
```

## Authorization

The minimum role for authorization is "administrator".

## Query parameters

| Query parameter | Type | Required? | Description | 
|:---|:---|:---|:---|
| usernames | array | required | list of usernames |

## Request Example

```curl
curl -X GET 'http://localhost/typemill/api/v1/users/getbynames?usernames[]=trendschau'
```

## Response Example

```json
{
  "userdata": [
    {
      "username": "myname",
      "firstname": "my",
      "lastname": "name",
      "email": "myname@xyz.net",
      "userrole": "member",
      "lastlogin": 1713889931,
      "apiaccess": false,
      "linkaccess": true
    },
    {
      "username": "yourname",
      "email": "yourname@xyz.com",
      "userrole": "administrator",
      "lastlogin": 1719432324,
      "firstname": "",
      "lastname": "",
      "description": "I have something to say",
      "image": "",
      "apiaccess": true,
      "darkmode": false,
      "authcodedata": "33121415513513413434225",
      "fingerprints": [
        "0f5113d4a2b4751f15a2ed787145978e",
        "6968aacfd1b6ccd3f9faa5db8260ebc5"
      ]
    }
  ]
}
```


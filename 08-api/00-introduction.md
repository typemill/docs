# Introduction

The Typemill API allows you to retrieve and update content and data using simple HTTP requests. It primarily powers the Typemill author interface, meaning you can perform all the same actions via the API that you would in the interface. Additionally, some extra endpoints have been introduced to support external use.

## Use Cases

Typemill is rarely used for headless content management or JAMstack integrations. Instead, it excels at creating documentation and manuals.  A great use case is integrating a documentation or knowledge base as inline help within another software. We have implemented this approach with the Typemill documentation itself, which also serves as the content repository for the inline help system in Typemill installations.

## Limitations 

Unlike traditional APIs that use fixed IDs to identify pages, Typemill relies on relative URLs. This works seamlessly within the author interface but can be a limitation for external integrations. If a page's URL changes, external systems relying on the old URL may no longer be able to find the content. To address this, we introduced an alternative endpoint that allows pages to be found using their slugs, which are more stable and less likely to change frequently.

## URL

The URL format is:

```
{protocol}://{domain}/api/{version}/{endpoint}
```

Example:

```
https://mytypemillwebsite.com/api/v1/navigation
```

## Activate the API

API calls are tied to user accounts. You can enable API access by checking a box in the user's profile settings. This can be done for multiple users as needed.

![Screenshot of the api checkbox in the user profile](media/live/api-authentication.webp){.center loading="lazy" width="820" height="301"}

## Authentication

To authenticate API calls, use **Basic Authentication** with the username and password of the respective user:

```http
curl -X GET "https://typemill.net/api/v1/navigation" \
     -u username:password \
     -H "Accept: application/json"
```

## Authorization

Permissions are determined by the user's role. Check the documentation about user roles for more details.

## Trusted IPs and Hosts

You can restrict API access based on IP addresses or hostnames, which can be configured in the Developer tab of the system settings. Check the [admin guide](/admin-guide/developer-tab) for more details.

## Recommendations

* Create separate user accounts for API access.
* Use clear and descriptive usernames for API user (e.g. with prefixes to easily identify them).
* Use strong passwords and the lowest role needed (e.g. with just read access).
* Allow API calls only from whitelisted IPs or hosts.

## CORS

Typemill does not support CORS headers, meaning the API cannot be accessed directly from the browser (e.g., using JavaScript). Only server-side calls are allowed. 

## Endpoints and Methods

| Endpoint | Method | Description | 
|:---|:---|:---|
| /navigation | get | Get the content navigation. | 
| /article | get | Get the content of a page in html and markdown format. | 
| /meta | get | Get the metadata (tabs) of a page. | 
| /item | get | Get the item for a relative url. | 
| /items | get | Get the item(s) for a slug. | 
| /pagemedia | get | Get a list of images and files used in the Markdown document of the page. | 
| /images | get | Get a list of all images in the media library. | 
| /files | get | Get a list of all files in the media library. | 
| /settings | get | Get the settings of the website. | 
| /users/getbyname | get | Get a list of users with a name or part of a name | 
| /users/getbymail | get | Get a list of users with an email or part of an email | 
| /users/getbyrole | get | Get a list of users with a certain userrole |

## HTTP Status Codes

The API will return the following status codes.

| Status code | Description | 
|---|---|
| 200 ok | The request was ok. | 
| 400 bad request | The request was unacceptable due to missing or invalid parameter. | 
| 401 unauthorized | The request requires an authorization. | 
| (402 request failed) | The parameters where there but the request failed for other reasons. | 
| 403 forbidden | The user is authenticated but he has not enough rights. | 
| 404 not found | The page or item was not found. | 
| 500 internal server error | An internal error by the application. | 


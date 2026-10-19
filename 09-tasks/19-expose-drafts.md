# expose drafts

## Summary

Typemill 2.27.0 does not enforce page ownership or page-level access controls on the author editor routes. An authenticated contributor can read unpublished draft content belonging to another user, including pages restricted by allowedrole or alloweduser.
Details

The routes /tm/content/raw/{path} and /tm/content/visual/{path} require only the mycontent:read permission. After resolving the requested path, the controllers load the draft markdown without applying the ownership and page-access checks used by the content API.

The missing check is in the web author controllers (ControllerWebAuthor::showRaw and ControllerWebAuthor::showBlox). The affected code was verified at commit dc644958c716ac393c63daeb98891c66eaae8ccd (Typemill 2.27.0).

## Proof of concept

* Create an unpublished page owned by alice and set its page restriction to allowedrole: manager.
* Log in as a different user with the contributor role (mallory).
* Request:
* `GET /tm/content/raw/<target-page-path>`
* or:
* `GET /tm/content/visual/<target-page-path>`
* The response is HTTP 200 and contains the target page's unpublished draft content.

This was reproduced against a local Typemill 2.27.0 instance. The test fixture was removed after verification.

## Impact

A contributor can read other authors' unpublished work and bypass page-level role/user restrictions. This can disclose confidential documentation, plans, or other pre-publication content.


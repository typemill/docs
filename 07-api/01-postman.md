# Testing with Postman

## Postman Collection

Download the zip-file with the Postman-collection and some helper files:

[Postman Testing Pack (GZ, 4.03 KB)](media/files/postmantar.gz){.tm-download file-gz}

The zip contains:

* `collection.json`: The postman collection that you can import into your postman workspace.
* `roles.json`:  The runner-data with roles.
* `scripts.js`: The pre-request-script for the collection and an example post-request test-script for an endpoint.
* `readme.md`: The copy of this documentation.

You can just use the collection, or setup a testing of all available roles in Typemill if you run a stage- or testing-environment. Please note that you have to create users with the described roles in your testing instance.

## Multi-role authorization testing in Postman (STAGE + Collection Runner)

This setup lets you run the same collection **multiple times (one iteration per role)** so you can validate **permissions and responses** across roles. It supports both:

* **Automated runs** via **Collection Runner** (data-driven)
* **Manual runs** via clicking **Send** on a single request (no iteration data)

Workspace entities:

- Environment: **STAGE** [STAGE](environment/52584569-8104f289-eadf-4107-b517-782bb43ce711)
- Collection: **Typemill API V1** [Typemill API V1](collection/52584569-13ff35c6-0fdb-4784-a3c4-40fb38d892cc)

### 1) Environment variables (STAGE)

In **STAGE**, we store one credential pair per role, plus “runtime” variables that the scripts set for the current iteration.

**Base**

* `BASE_URL` :  Base URL of the stage system (used in request URLs).

**Per-role credentials (static)**

* `BASIC_USER_apiguest`
* `BASIC_PASS_apiguest`
* `BASIC_USER_apimember`
* `BASIC_PASS_apimember`
* `BASIC_USER_apicontributor`
* `BASIC_PASS_apicontributor`
* `BASIC_USER_apiauthor`
* `BASIC_PASS_apiauthor`
* `BASIC_USER_apieditor`
* `BASIC_PASS_apieditor`
* `BASIC_USER_apimanager`
* `BASIC_PASS_apimanager`
* `BASIC_USER_apiadmin`
* `BASIC_PASS_apiadmin`

**Runtime variables (set by script)**

- `BASIC_USER` (current username for the active iteration)
- `BASIC_PASS` (current password for the active iteration)
- `ACTIVE_ROLE` (current role name, e.g. `apiguest`)

Notes:

- Mark passwords as **sensitive** if desired.
- `BASIC_USER` / `BASIC_PASS` can be empty initially; they get set automatically during runs.

### 2) Collection auth configuration

On the collection, set:

- **Auth type:** Basic Auth  
- **Username:** `{{BASIC_USER}}`  
- **Password:** `{{BASIC_PASS}}`

This ensures the collection uses the **runtime** credentials, which change per iteration.

### 3) Runner iteration data (roles file)

To run the collection across all roles, use Collection Runner with a JSON data file like:

```json
[
  { "role": "apiguest" },
  { "role": "apimember" },
  { "role": "apicontributor" },
  { "role": "apiauthor" },
  { "role": "apieditor" },
  { "role": "apimanager" },
  { "role": "apiadmin" },
]
```

Rules:

* The `role` value must match the suffix used in your env vars: role `apiguest` → `BASIC_USER_apiguest` / `BASIC_PASS_apiguest`
* You can add/remove roles by editing this list.
* You can extend each row with extra fields later (e.g. `expectedStatus`, `note`, etc.).

### 4) Collection-level Pre-request Script

This script handles the role → credentials mapping. Add this **once at the collection level** so it runs before every request.

Purpose:

- Decide which role is active (Runner iteration or manual)
- Load the corresponding username/password from environment variables
- Set `BASIC_USER` / `BASIC_PASS` for collection auth
- Set `ACTIVE_ROLE` for test assertions and reporting

```javascript
let role = pm.iterationData.get('role');

// Allow manual "Send" without the Runner
if (!role) {
  role = pm.environment.get("ACTIVE_ROLE"); // set manually when needed
}

// Optional default (pick what makes sense for you)
if (!role) {
  role = "apiauthor";
}

const userKey = `BASIC_USER_${role}`;
const passKey = `BASIC_PASS_${role}`;

const user = pm.environment.get(userKey);
const pass = pm.environment.get(passKey);

if (!user || !pass) {
  throw new Error(`Missing env vars for role '${role}'. Expected ${userKey} and secret passKey`);
}

pm.environment.set('BASIC_USER', user);
pm.environment.set('BASIC_PASS', pass);
pm.environment.set('ACTIVE_ROLE', role);

// for logging
pm.test(`Role used: ${role}`, () => {
  pm.expect(role).to.be.a("string");
});

console.log("Auth role:", role, "userKey:", userKey);
```

Manual usage:

- Set `ACTIVE_ROLE` in STAGE to the role you want (e.g. `apimanager`)
- Click **Send** on any request; it will authenticate as that role

Runner usage:

- `ACTIVE_ROLE` is set automatically each iteration from the data file

### 5) Per-endpoint Tests Post-response Script

Each endpoint should have tests that verify permission behavior by role.

Typical pattern:

```javascript
const role = pm.environment.get("ACTIVE_ROLE");

// Define which roles are allowed for *this endpoint*
const allowedRoles = ["apiadmin"]; // customize per request

pm.test("Permission matches role", () => {
  if (allowedRoles.includes(role)) {
    pm.expect(pm.response.code).to.be.oneOf([200, 201, 204]);
  } else {
    pm.expect(pm.response.code).to.be.oneOf([401, 403]);
  }
});
```

Guidelines:

- Put the permission matrix **close to the endpoint** (easy to maintain).
- For “read” endpoints, `allowedRoles` may be larger.
- For “write/admin” endpoints, restrict it.
- If your API returns `404` to hide resources from unauthorized users, include that in the disallowed list for that endpoint.

Optional: make the role visible in test output:

```javascript
pm.test(`Role used: ${role}`, () => {
  pm.expect(role).to.be.a("string");
});
```

### 6) Suggested operational workflow

**To run permission regression across all roles**

1. Select environment: **STAGE**
2. Open the collection runner for **Typemill API V1**
3. Attach (upload or select) the roles JSON file
4. Run the collection
5. Review failures per iteration (role) and per request

**To debug one endpoint manually**

1. Set `ACTIVE_ROLE` in **STAGE** to a role (e.g. `apiadmin`)
2. Open the request and click **Send**
3. Confirm the response and tests

### 7) Troubleshooting checklist

- All iterations returning the same result:
  - Collection auth must be `{{BASIC_USER}}/{{BASIC_PASS}}` (not a role-specific variable)
  - Ensure request-level auth is not overriding the collection
- Manual Send fails with “Iteration data must include role”:
  - Use the “manual fallback” version of the pre-request script (above)
  - Set `ACTIVE_ROLE` in STAGE before sending
- Unexpected `302` responses:
  - Often indicates redirect-to-login or missing auth
  - Confirm the auth is applied and check response headers (Location)


# Typemill Docker Proxy Detection – Implementation Plan

## Background

Typemill is being prepared as a container template for the mittwald mStudio.

The mittwald local development environment runs applications behind a Caddy reverse proxy:

```text
Browser
  ↓ HTTPS
Caddy
  ↓ HTTP + X-Forwarded-* headers
Typemill container
  ↓
Apache / PHP
```

The Typemill Docker container itself serves HTTP on port 80. Caddy terminates TLS and forwards the request to Typemill.

During local testing, Typemill generated `http://` URLs for CSS/JS assets instead of `https://` URLs. Firefox consequently blocked some resources as mixed content.

Typemill already has proxy detection support. It needs to be explicitly enabled for Typemill to use the reverse-proxy headers.

## Precheck Typemill Implementations

1. Check why the proxy detection must be activated manually and does not work automatically, I guess it has security reasons, because with automatic proxy detection a user can change the x-forwarded header and do something bad. Confirm that the proxy setting is `false` by default and that the trusted-IP check in `Urlinfo::checkIp()` and the RKA `ProxyDetection` middleware provide the current security boundary.
2. It is probably not relevant, but the proxy detection still lives in middleware, while the urlinfo is created when the script boots. It might be an idea to integrate the proxy detection into the urlinfo and remove it from middleware to avoid duplications, but that might have drawbacks. Please investigate.
   - `system/typemill/Static/Urlinfo.php` already runs proxy detection during boot to generate `urlinfo` (used by assets, license, Twig, session segments, etc.).
   - `system/typemill/system.php` also adds the RKA `ProxyDetection` middleware to the Slim request pipeline so controllers receive an updated PSR-7 request object.
   - For this task, do **not** refactor or remove either mechanism. The goal is only to enable the existing proxy detection path for Docker via an environment variable. A future cleanup can consolidate the duplication.
3. Check if there is a better way to solve the docker problem with proxy detection than the current implementation plan suggests.
   - The preferred solution is an environment variable evaluated at runtime by the Settings model, because it does not touch `settings.yaml`, works before the first setup, and is reversible without image changes.

## Relevant Typemill logic

Routing depends on the `setup` setting:

```php
if(isset($settings['setup']) && $settings['setup'] == true)
{
    require __DIR__ . '/routes/setup.php';
}
else
{
    require __DIR__ . '/routes/api.php';
    require __DIR__ . '/routes/web.php';
}
```

Default settings are loaded from `defaults.yaml`, while user settings are loaded from `settings.yaml`:

```php
// system/typemill/Models/Settings.php
public function getDefaultSettings()
{
    $defaultSettings = $this->storage->getYaml('systemSettings', '', 'defaults.yaml');

    if($defaultSettings)
    {
        $defaultSettings['systemSettingsPath'] = $this->storage->getFolderPath('systemSettings');

        return $defaultSettings;
    }

    return false;
}

public function getUserSettings()
{
    $userSettings = $this->storage->getYaml('settingsFolder', '', 'settings.yaml');

    if($userSettings)
    {
        return $userSettings;
    }

    return false;
}
```

The current `loadSettings()` logic initially sets:

```php
// system/typemill/Models/Settings.php
$settings = $defaultsettings;
$settings['setup'] = true;
```

When user settings exist:

```php
$settings = array_merge($defaultsettings, $usersettings);
```

This removes `setup: true` unless it is explicitly present in the user settings. This is intentional and controls whether the setup routes are used.

## Important constraints

- Do **not** add `proxy: true` to the normal `defaults.yaml`.
- Proxy detection should remain opt-in for normal Typemill installations.
- Do **not** create or modify `settings.yaml` from the Docker entrypoint to enable proxy detection.
- Existing user settings must never be overwritten during container startup or upgrades.
- The normal Typemill setup/registration flow must continue to work.
- Do not refactor the Urlinfo/ProxyDetection duplication as part of this change; keep the scope minimal.

The Docker environment is the special case because the application is expected to run behind a reverse proxy.

---

# Implementation

## 1. Dockerfile

Edit: `/var/www/html/typemill/Dockerfile`

Add after the base image and dependency installation steps:

```dockerfile
ENV TYPEMILL_PROXY_DETECTION=true
```

This makes proxy detection enabled by default for the Docker image.

It can be overridden at runtime:

```yaml
environment:
  TYPEMILL_PROXY_DETECTION: "false"
```

No changes should be made to `defaults.yaml`.

No changes to `settings.yaml` should be performed by `init-server`.

---

## 2. Typemill Settings Model

Edit: `/var/www/html/typemill/system/typemill/Models/Settings.php`

Modify `loadSettings()` so that the environment variable is evaluated **after the user settings have been merged**.

Conceptually:

```php
public function loadSettings()
{
    $defaultsettings = $this->getDefaultSettings();
    $usersettings = $this->getUserSettings();

    $settings = $defaultsettings;
    $settings['setup'] = true;

    if($usersettings)
    {
        $settings = array_merge($defaultsettings, $usersettings);

        # make sure all image-size information are there
        if(isset($usersettings['images']))
        {
            $images = array_merge($defaultsettings['images'], $settings['images']);
            $settings['images'] = $images;
        }
    }

    if(getenv('TYPEMILL_PROXY_DETECTION') === 'true')
    {
        $settings['proxy'] = true;
    }

    $settings['rootPath'] = getcwd();

    $settings = self::addThemeSettings($settings);

    return $settings;
}
```

Requirements:

- The proxy override must happen **after** `array_merge($defaultsettings, $usersettings);` so that the Docker environment setting cannot disappear because of the merge.
- The override must happen **before** `addThemeSettings()` and the `OnSettingsLoaded` event are dispatched.
- Use a strict comparison: `getenv('TYPEMILL_PROXY_DETECTION') === 'true'`. Only the exact string `true` enables proxy detection. Other values (`1`, `yes`, `TRUE`, empty string, unset) are ignored and leave the setting at its default.
- The setting does not need to be persisted in `settings.yaml`.

### Expected behavior

| Installation | Environment | User setting | Effective `proxy` |
|---|---|---|---|
| New Docker install | `TYPEMILL_PROXY_DETECTION=true` | none | `true` |
| Existing Docker install | `TYPEMILL_PROXY_DETECTION=true` | none | `true` |
| Existing Docker install | `TYPEMILL_PROXY_DETECTION=true` | `proxy: false` | `true` (env overrides) |
| Docker install | `TYPEMILL_PROXY_DETECTION=false` | none | default/off |
| Normal Typemill | variable absent | none | default/off |
| Normal Typemill | variable absent | `proxy: true` | `true` |
| Normal Typemill | `TYPEMILL_PROXY_DETECTION=true` | none | `true` (env can also be used outside Docker, but still opt-in at runtime) |

The exact precedence should be confirmed during review. The proposed implementation treats the Docker environment variable as the runtime override for the current process.

Most importantly, an existing `settings.yaml` is never modified by this change.

---

# Why not modify `settings.yaml`?

An earlier approach considered adding:

```yaml
proxy: true
```

to `settings.yaml` from `init-server`.

This should **not** be implemented.

Creating a user `settings.yaml` during container initialization could interfere with Typemill's initial setup detection.

Runtime injection into the loaded settings avoids this problem completely.

---

# Docker initialization

`/var/www/html/typemill/docker-utils/init-server` should remain responsible for:

- populating persistent content directories
- populating persistent themes
- populating persistent media
- initializing the settings volume from `settings.default`
- fixing ownership
- starting Apache

It should **not** modify `settings.yaml` to enable proxy detection.

The existing settings initialization can remain:

```sh
if [ -z "$(ls -A /var/www/html/settings)" ]; then
    cp -R /var/www/html/settings.default/* /var/www/html/settings/
fi
```

No proxy-specific logic is required in `init-server`.

---

# Risks and mitigations

| Risk | Mitigation |
|---|---|
| Environment variable leaks to non-Docker users | Only exact string `true` triggers it; it remains opt-in at runtime. |
| Existing user setting `proxy: false` is overridden | Acceptable for Docker, because the environment variable represents the deployment contract. Document this behavior. |
| `getenv()` is disabled in some hosting environments | Docker is the target; normal installations use the admin UI checkbox. |
| `settings.yaml` is overwritten | The change is read-only; no write operations are added. |
| Setup detection (`setup: true`) is affected | The proxy override runs after the merge but does not alter `setup`, so first-run setup still works. |
| Middleware/Urlinfo duplication grows | Out of scope; note for future cleanup. |

---

# Testing Plan

When you test, NEVER delete any core files of Typemill. Only delete files that you have created during your tests.

## A. Normal Typemill setup (non-Docker)

Test the PHP change without Docker first.

Verify:

1. A fresh installation still reaches the Typemill setup/registration page.
2. `setup: true` is still present during the initial setup.
3. Completing setup creates the normal `settings.yaml`.
4. Subsequent requests no longer use the setup routes.
5. No unexpected `proxy` setting is written to `settings.yaml`.
6. Setting `proxy: true` in the admin UI still works as before.

## B. Docker / mittwald local environment

Stop local Apache because the mittwald development proxy needs ports 80/443:

```bash
sudo systemctl stop apache2
```

Start the template:

```bash
pnpm app typemill
```

Open:

```text
https://typemill.localhost
```

Verify:

- Typemill loads over HTTPS.
- CSS loads without mixed-content errors.
- JavaScript/Vue loads correctly.
- URLs generated by Typemill use HTTPS.
- Initial Typemill setup works.
- Completing setup creates the normal user settings.
- Restarting the container retains proxy detection.

## C. Environment override

Test:

```yaml
environment:
  TYPEMILL_PROXY_DETECTION: "false"
```

Verify that proxy detection is disabled.

Then test the default:

```text
TYPEMILL_PROXY_DETECTION=true
```

and verify HTTPS URL generation again.

Also test edge cases:

- `TYPEMILL_PROXY_DETECTION=true` → proxy enabled.
- `TYPEMILL_PROXY_DETECTION=false` → proxy disabled.
- `TYPEMILL_PROXY_DETECTION=1` → proxy disabled (strict string check).
- `TYPEMILL_PROXY_DETECTION=TRUE` → proxy disabled.
- variable absent → follows `settings.yaml` or default.

## D. Existing installation

Start with an existing `settings.yaml`.

Verify that:

- the file is not modified by container startup
- existing settings remain intact
- proxy detection is enabled when the environment variable is `true`
- restarting/upgrading the container does not overwrite user settings

## E. Verify headers

With a running request and the mittwald proxy:

```bash
curl -I https://typemill.localhost
```

Verify that the `X-Forwarded-Proto`, `X-Forwarded-Host`, and `X-Forwarded-Port` headers are present in the request as seen by the container.

Check that Typemill generates asset URLs with `https://` and not `http://`.

---

# Local mittwald environment

The mittwald development proxy uses Caddy and can remain running after:

```bash
pnpm app typemill down
```

After testing, check:

```bash
docker ps
```

If these shared development containers are still running:

```text
ct-dev-proxy-caddy-1
ct-dev-proxy-mail-1
```

stop them:

```bash
docker stop ct-dev-proxy-caddy-1 ct-dev-proxy-mail-1
```

Verify that ports 80/443 are free:

```bash
sudo ss -ltnp | grep -E ':80 |:443 '
```

Then restart normal Apache:

```bash
sudo systemctl start apache2
```

Check:

```bash
sudo systemctl status apache2 --no-pager
```

To start the mittwald test environment again:

```bash
sudo systemctl stop apache2
pnpm app typemill
```

---

# Release implications

The change affects Typemill's PHP code and the Docker image.

After successful testing, the developer (NOT the opencode agent) should:

1. Commit the Settings Model change.
2. Add `ENV TYPEMILL_PROXY_DETECTION=true` to the Dockerfile.
3. Create a new Typemill release/version.
4. Build and publish the corresponding Docker image.
5. Update the mittwald container template to use the new image version.
6. Run the mittwald repository validation:

```bash
pnpm test
```

Expected validation result:

```text
0 error(s)
```

The mittwald template itself does not need to implement proxy detection logic; the behavior belongs in the Typemill Docker image.

---
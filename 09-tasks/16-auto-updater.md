# Typemill Update & Installation 

## Requirements

### One simple installation mechanism

* Download package
* Verify SHA-256
* Extract to temporary location
* Validate
* Install/replace
* Clean up

### Core, plugins and themes

* The same mechanism should be usable for all three (core, plugins, themes).
* Core needs a new version-specific download mechanism.
* Plugins/themes already have download endpoints.

### Updates

* Use the existing version checker.
* Detect newer plugin/theme/core versions.
* Pass each update to the common installer.

### Secure downloads

* Downloads must refer to an exact version.
* API provides the expected SHA-256 hash.
* Installation only proceeds when the hash matches.

### Hosting compatibility

* Prefer cURL for downloads.
* Fall back to PHP streams where possible.
* Do not rely exclusively on `file_get_contents()`.
* Optionally support manual ZIP upload for servers without outbound HTTP access.

### Safe installation

* Never extract directly into the live installation.
* Use a temporary/staging directory.
* Back up the existing component before replacement.
* Restore the previous version if installation fails.

### Future marketplace and readymades

* Admin can list/search available plugins and themes.
* Clicking **Install** uses exactly the same installation mechanism as updates.
* A readymade installation is simply a definition containing:
     * theme(s)
     * plugin(s)
     * optionally content
* Typemill installs the listed components using the same mechanism.

### Keep it simple

* No full package manager.
* No complex dependency resolver initially.
* No separate mechanisms for updates, marketplace installation, and readymade installations.
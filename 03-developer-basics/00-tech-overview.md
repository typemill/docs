# Technology Overview

Typemill uses the following technologies for its core application:

* **PHP**: Required versions and extensions are outlined in the [Getting Started](/getting-started) guide.
* **Slim Framework 4**: A minimal PHP framework that provides routing and basic structure. You can find the [slim documentation here](https://www.slimframework.com/).
* **Twig Templates**: Used for rendering templates with php in the backend.
* **Vue.js 3**: Powers the entire admin interface. The outer frame is rendered with Twig, but all inner content is managed by Vue.js. Typemill uses the *Options API* (not the Composition API).
* **Tailwind CSS 4**: Used for styling the admin interface. Most themes, however, use Tachyons CSS.
* **Tachyons CSS**: A functional CSS toolkit used by most default themes. If you create your own themes, you can use any CSS framework or custom styles.

## Additional Libraries

Beyond the core stack, Typemill uses several important libraries:

* **Parsedown** and **Parsedown Extra**: For converting Markdown to HTML.
* **Symfony Event Dispatcher**: Provides the event system.
* **Laminas ACL**: Used for managing access control and permissions.
* **Valitron**: A simple validation library for user inputs.

## Dependency Management

Typemill uses **Composer** to manage all PHP dependencies.

To keep things simple, Typemill does not use JavaScript package managers like **npm or yarn**, and it does not use build tools like **webpack or vite** (except for Tailwind CSS). Instead, all frontend resources are added the traditional way – directly via plain JavaScript or CSS files without a build process.


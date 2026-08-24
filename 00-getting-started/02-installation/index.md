# Install Typemill in 5 Minutes

You can install Typemill manually, via GitHub, or with Docker.

## Manual Installation

1. **Download** [Typemill](https://typemill.net/starter-guide/download) or a specific bundle for your use case.
2. Ensure you have an **Apache** or **Nginx** server with **PHP 8.2** or higher.
3. **Unzip** Typemill and **upload** the files to your server.
4. Visit your website and **create an admin user**.
5. Login with your admin under `/tm/login` and start writing immediately.

Don't forget to check the [permission](/getting-started/installation/permissions) for folders and files and the required [php libraries](/getting-started/installation/php). You can also test Typemill on your [local machine](/getting-started/installation/localhost). The [troubleshooting guide](/getting-started/installation/troubleshooting) will help you if you run into any problems.

## Installation via GitHub

Clone Typemill from [GitHub](https://github.com/typemill/typemill):

```bash
git clone "https://github.com/typemill/typemill.git"
```

Install dependencies with Composer (make sure you have Composer installed):

```bash
composer install
```

If you prefer to update all libraries to their latest versions, you can use `composer update` instead.

## Installation with Docker

Pull Typemill from [DockerHub](https://hub.docker.com/r/kixote/typemill):

```bash
docker pull kixote/typemill
```

Start a Typemill container:

```bash
docker run -d --name typemill -p 8080:80 kixote/typemill
```

Typemill will now be available at `http://localhost:8080`. You can also build your own [Docker image locally](/getting-started/installation/docker).

## TLS and Proxy Detection

The Docker image in the Typemill repository does not provide TLS support. It's intended for local use or for running behind a reverse proxy.

Proxy detection is **enabled by default** in the Docker image. This allows Typemill to correctly detect the original protocol and host when running behind a reverse proxy such as Nginx, Caddy, Traefik, or Cloudflare. The image uses the `X-Forwarded-*` headers provided by the proxy.

You can disable proxy detection with the `TYPEMILL_PROXY_DETECTION` environment variable:

```yaml
environment:
  TYPEMILL_PROXY_DETECTION: "false"
```


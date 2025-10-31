#  Run Typemill with Docker

The Typemill repository contains a Dockerfile that helps you create your own Docker image locally. The Dockerfile is a community contribution by [Matthieu Borgognon](https://github.com/matbgn). You can also use the [official image on DockerHub](https://hub.docker.com/r/kixote/typemill) or try any [community image](https://hub.docker.com/search?q=Typemill).

! **Note:** The Docker image in the Typemill repository does not provide TLS support. It's perfect either for local use or behind your own proxy.

## Local Setup

Clone the Typemill repository:

```
git clone git://github.com/trendschau/typemill.git
cd typemill
```

Build your image locally:

```
docker build -t typemill:local .
```

Run the Docker image without persistence on port 8080:

```
docker run -d --name typemill -p 8080:80 typemill:local
```

Run Typemill with persistence:

```bash
docker run -d \
    --name=typemill \
    -p 8080:80 \
    -v $(pwd)/typemill_data/settings/:/var/www/html/settings/ \
    -v $(pwd)/typemill_data/media/:/var/www/html/media/ \
    -v $(pwd)/typemill_data/data/:/var/www/html/data/ \
    -v $(pwd)/typemill_data/cache/:/var/www/html/cache/ \
    -v $(pwd)/typemill_data/plugins/:/var/www/html/plugins/ \
    -v $(pwd)/typemill_data/content/:/var/www/html/content/ \
    -v $(pwd)/typemill_data/themes/:/var/www/html/themes/ \
    typemill:local
```

A simple `docker-compose.yml` file could look like this:

```yml
version: "2.0"

services:
  typemill:
    image: typemill:local
    volumes:
      - ./typemill_data/settings/:/var/www/html/settings/
      - ./typemill_data/media/:/var/www/html/media/
      - ./typemill_data/data/:/var/www/html/data/
      - ./typemill_data/cache/:/var/www/html/cache/
      - ./typemill_data/plugins/:/var/www/html/plugins/
      - ./typemill_data/content/:/var/www/html/content/
      - ./typemill_data/themes/:/var/www/html/themes/
    ports:
      - 8080:80
```

## Volumes

* `settings`: persists user profiles, site configuration, etc. (empty by default)
* `media`: persists media files (empty by default)
* `data`: persists data like cached navigation and stored data from plugins (empty by default)
* `cache`: persists cache files for performance purposes (optional and empty by default)
* `plugins`: persists installed plugins (optional and empty by default)
* `content`: persists published content (will be initialized with default examples if the bound volume is empty)
* `themes`: persists installed themes (will be initialized with default examples if the bound volume is empty)


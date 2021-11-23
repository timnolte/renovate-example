FROM forumone/wordpress:7.4-xdebug AS dev

FROM forumone/wordpress:7.4 AS base

FROM composer:2 AS composer

RUN mkdir -p public

COPY ["auth.json", "composer.json", "composer.lock", "./"]

RUN set -ex \
  && composer install --no-dev --optimize-autoloader

# Install additional dev dependencies for the test image.

FROM composer AS composer-dev

RUN set -ex \
  && composer install --optimize-autoloader

FROM forumone/gesso:php7.3-node12 AS gesso

# Install npm dependencies

COPY ["public/wp-content/themes/gesso/package*.json", "./"]

RUN if test -e package-lock.json; then npm ci; else npm i; fi

# Copy sources and build

COPY ["public/wp-content/themes/gesso", "./"]

RUN set -ex \
  && gulp build

# Use a temporary image to clean dev dependencies for production. This allows

# the gesso-dev stage to start with these files in place rather than rebuilding.

FROM gesso AS gesso-clean

RUN set -ex \
  && rm -rf node_modules

# Install all dev dependencies for the test image.

FROM gesso AS gesso-dev

RUN set -ex \
  && npm install

FROM base AS release

# Copy all artifacts into the production-ready release stage for the final image.

COPY --from=composer ["/app/public", "public"]

COPY --from=gesso-clean ["/app", "public/wp-content/themes/gesso"]

COPY ["public", "public"]

RUN if test -e .env.production; then mv .env.production .env; fi

FROM release AS test

# Copy all dev dependencies into a stage for a testing image.

COPY --from=composer-dev ["/app/public", "public"]

COPY --from=gesso-dev ["/app", "public/wp-content/themes/gesso"]

# Ensure the default image to be built is the release image so any builds

# not explicitly defining a target receive the production release image

FROM release

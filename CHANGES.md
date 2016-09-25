# Changelog

## v0.4.0

- Fixes broken `dup build`
- Makes `dup status` actually check the result of `docker inspect` to ensure the container is actually running

## v0.3.12

Changed the `env` build argument to be `develop` to match Studio None's internal infrastructure.

Added `buildArgs` option to `.up.json`, follows the same format as `env` but is passed as a `--build-arg name=value` pair when `dup build` is executed.

## v0.3.11

This release fixes the issues with orphan containers caused by a missing flag in the `docker rm` command, run with `dup down`.

This release also adds basic support for `dup status`, to check whether the web and database containers are running.

## v0.3.10

This release added support for the official PostgreSQL Docker image, to allow the usage of version 9.5 (for it's `jsonb` support). In the future, you will be able to specify which point release of PostgreSQL your project needs.

## v0.3.7

The main new feature in v0.3.7 is the addition of `--build-arg env=dev` to the `dup build` command. This is a temporary measure until I expose build-args to the user via the `.up.json` file. This feature means you **need** to have this line in your base `Dockerfile`:

```Dockerfile
ARG env
```

This is the bare minimum. You will likely want to instead do the following:

```Dockerfile
ARG env=prod
```

This will default the argument to `prod`, and fix the `One or more build-args [env] were not consumed, failing build.` error that will otherwise be triggered. An example of how to leverage this feature to allow for a singular production `Dockerfile` that has developer tools added on top, assuming two files: `conf/start.prod.sh` which is the standard entry-point, and `conf/start.dev.sh` which is the development entry-point that adds extra tools is as follows:

```Dockerfile
###
# This is both a test of Skate as a tool, as well as build-args for Docker
##
FROM studionone/node6:latest

# Use a different entrypoint for development vs production
ARG env=prod
ADD conf/start.$env.sh /start.sh
RUN chmod +x /start.sh

ADD code /app
WORKDIR /app
RUN npm install

EXPOSE 8080

ENTRYPOINT /start.sh
```
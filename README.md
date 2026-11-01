# uv-docker
Testing using docker to run python file using uv for development.

## Dockerfile Breakdown
```dockerfile
FROM ghcr.io/astral-sh/uv:python3.15-trixie-slim AS builder
ENV UV_COMPILE_BYTECODE=1 UV_LINK_MODE=copy
```

Pulls the Python 3.15 image based on Debian Trixie (13) that includes uv. `AS builder` allows instructions to refer to this image.
`UV_COMPILE_BYTECODE`compiles the python file into bytecode at install time, install will take longer and the file will be bigger, but the final runtime will be quicker. `UV_LINK_MODE` ignores warning about different filesystems for the cache and the target.

```dockerfile
ENV UV_NO_DEV=1
ENV UV_PYTHON_DOWNLOADS=0
```
`UV_NO_DEV` ignores the development only packages. `UV_PYTHON_DOWNLOADS` disables python downloads as we will be using the built-in interpreter.

```dockerfile
WORKDIR /app
RUN --mount=type=cache,target=/root/.cache/uv \
    --mount=type=bind,source=uv.lock,target=uv.lock \
    --mount=type=bind,source=pyproject.toml,target=pyproject.toml \
    uv sync --locked --no-install-project
COPY . /app
RUN --mount=type=cache,target=/root/.cache/uv \
    uv sync --locked
```
`WORKDIR /app` sets the name of the directory the folder will use.
The large `RUN` command is described as below, the backslash is a simple end-of-line indicator:
|Command|Description|
|-|-|
|--mount=type=cache,target=/root/.cache/uv|Mounts a temp directory for a cache, and specifies where it is within the container|
|--mount=type=bind,source=uv.lock,target=uv.lock|Specifies a read-only directory (or file in this case), copying our `uv.lock` to the image `uv.lock`|
|--mount=type=bind,source=pyproject.toml,target=pyproject.toml|Specifies a read-only directory (or file in this case), Copying our `pyproject.toml` to the image `pyproject.toml`|
|uv sync --locked --no-install-project|Installs any required dependencies, checks the lockfile is upt-to-date, and doesn't install the project|

## Sources
All of the sources used for understanding the code.
### uv
Sources relating to uv:
#### Official
- [Docker documentation](https://docs.astral.sh/uv/guides/integration/docker/#developing-in-a-container)
- [Documentation for syncing projects](https://docs.astral.sh/uv/concepts/projects/sync/)
- [Example multi-stage dockerfile](https://github.com/astral-sh/uv-docker-example/blob/main/multistage.Dockerfile)
- [Docker example](https://github.com/astral-sh/uv-docker-example)

#### External
- [Extra description for how to implement uv in Docker](https://pythonresources.com/guides/docker-for-python-with-uv/)

### Docker
Sources relating to Docker:
#### Official
- [Dockerfile documentation](https://docs.docker.com/reference/dockerfile)

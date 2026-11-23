# syntax=docker/dockerfile:1
# Build the locked environment with uv, then copy only the environment and
# the code into a slim runtime image that runs as an unprivileged user.
# Base images are pinned by digest; update them deliberately.

FROM python:3.15.2-slim-trixie@sha256:65a65fc7b30b06cb6df4930fd4650697cb23f4c2698ab81fc87e4b031bc36928 AS build
COPY --from=ghcr.io/astral-sh/uv:0.13.1@sha256:4a1f3666816dafadd20ceb71eac9a7d56d7ef30279f7d64048519ab901ada336 /uv /bin/uv
ENV UV_COMPILE_BYTECODE=1 \
    UV_LINK_MODE=copy \
    UV_PYTHON_DOWNLOADS=never \
    UV_PROJECT_ENVIRONMENT=/opt/venv
WORKDIR /src
COPY pyproject.toml uv.lock .python-version ./
RUN --mount=type=cache,target=/root/.cache/uv uv sync --locked --no-dev
COPY alembic.ini ./
COPY app ./app
COPY migrations ./migrations

FROM python:3.15.2-slim-trixie@sha256:65a65fc7b30b06cb6df4930fd4650697cb23f4c2698ab81fc87e4b031bc36928 AS runtime
# pip installs packages, which the running image never does; without it the
# image carries none of the libraries pip vendors, so their advisories do not
# fail the build's scan.
RUN useradd --system --uid 10001 --no-create-home app \
  && python -m pip uninstall --yes --quiet --root-user-action ignore pip
COPY --from=build /opt/venv /opt/venv
COPY --from=build /src /srv/app
WORKDIR /srv/app
ENV PATH=/opt/venv/bin:$PATH \
    PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_HOST=0.0.0.0 \
    APP_PORT=8000
USER 10001
EXPOSE 8000
HEALTHCHECK --interval=30s --timeout=3s --start-period=10s \
  CMD ["python", "-c", "import sys, urllib.request; sys.exit(0 if urllib.request.urlopen('http://127.0.0.1:8000/health/live', timeout=2).status == 200 else 1)"]
CMD ["python", "-m", "app"]

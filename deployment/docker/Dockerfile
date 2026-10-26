# syntax=docker/dockerfile:1
# Build from repository root:
#   docker build -f deployment/docker/Dockerfile .

FROM oven/bun:1.3.14 AS build

WORKDIR /app

COPY package.json bun.lock ./
RUN bun install --frozen-lockfile

COPY . .

ARG VERSION=dev
ENV VERSION=${VERSION}

RUN bun run build \
  && mv dist/index.html dist/management.html \
  && cp dist/management.html dist/index.html

FROM nginx:1.27-alpine

LABEL org.opencontainers.image.title="CLI Proxy API Management Center" \
  org.opencontainers.image.description="Official web UI for CLIProxyAPI with optional API reverse proxy" \
  org.opencontainers.image.source="https://github.com/router-for-me/Cli-Proxy-API-Management-Center"

ENV NGINX_PORT=8080 \
  NGINX_RESOLVER=127.0.0.11 \
  CLIPROXY_API_BACKEND=http://cli-proxy-api:8317

COPY --from=build /app/dist/management.html /usr/share/nginx/html/management.html
COPY --from=build /app/dist/index.html /usr/share/nginx/html/index.html
COPY deployment/docker/nginx.conf.template /etc/nginx/templates/default.conf.template

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD wget -qO- "http://127.0.0.1:${NGINX_PORT}/management.html" >/dev/null 2>&1 || exit 1

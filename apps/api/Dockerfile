# Alpine image
FROM node:20-alpine AS base

ENV PNPM_HOME="/pnpm"
ENV PATH="$PNPM_HOME:$PATH"
RUN apk update && apk add --no-cache \
  libc6-compat \
  build-base \
  gcc \
  autoconf \
  automake \
  zlib-dev \
  libpng-dev \
  nasm \
  bash \
  vips-dev \
  git

RUN corepack enable && \
	corepack prepare --activate pnpm@latest && \
	pnpm config -g set store-dir /pnpm/store && \
  npm install -g \
    turbo \
    node-gyp

WORKDIR /pkrbt

FROM base AS build

COPY pnpm-lock.yaml /pkrbt
RUN pnpm fetch --prod --filter=api
COPY . /pkrbt

WORKDIR /pkrbt

RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --filter=api --frozen-lockfile
RUN turbo prune --docker @pkrbt/api
RUN pnpm deploy --filter=api /prod/api

WORKDIR /prod/api
RUN pnpm build

FROM base AS prod

COPY --from=build --chown=node:node /prod/api /api
USER node
WORKDIR /api
EXPOSE 1337

CMD [ "pnpm", "start" ]

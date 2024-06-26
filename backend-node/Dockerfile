# use the official Bun image
# see all versions at https://hub.docker.com/r/oven/bun/tags
ARG NODE_VERSION=20.9

FROM node:${NODE_VERSION}-alpine as base

WORKDIR /usr/src/app

# install dependencies into temp directory
# this will cache them and speed up future builds
FROM base AS install
RUN mkdir -p /temp/dev
COPY package.json yarn.lock /temp/dev/
RUN cd /temp/dev && yarn install --frozen-lockfile

# install with --production (exclude devDependencies)
RUN mkdir -p /temp/prod
COPY package.json yarn.lock /temp/prod/
RUN cd /temp/prod && yarn install --frozen-lockfile --production

# copy node_modules from temp directory
# then copy all (non-ignored) project files into the image
FROM base AS prerelease
COPY --from=install /temp/dev/node_modules node_modules
COPY . .

# [optional] tests & build
ENV NODE_ENV=production
# RUN yarn test
RUN yarn build

# copy production dependencies and source code into final image
FROM base AS release
COPY --from=install /temp/prod/node_modules node_modules
COPY --from=prerelease /usr/src/app .
COPY --from=prerelease /usr/src/app/package.json .
COPY --from=prerelease /usr/src/app/.env.production ./build/.env
COPY --from=prerelease /usr/src/app/.env.production .env

# run the app
EXPOSE 3000
ENTRYPOINT [ "node", "./build/bin/server.js" ]
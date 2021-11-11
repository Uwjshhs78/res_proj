#############
### base ###
#############
FROM node:14-alpine as base

#The name of the app to build
ARG app
ENV APP=$app
RUN echo building ${APP}
#############
### build ###
#############
FROM base as build

WORKDIR /app
# add `/app/node_modules/.bin` to $PATH
ENV PATH /app/node_modules/.bin:$PATH

# install nrwl cli
RUN npm install -g @nrwl/cli

# copy dependencies
# Copy over dependency list
COPY tsconfig.base.json /app/tsconfig.base.json
COPY package.json /app/package.json
COPY package-lock.json /app/package-lock.json
COPY declarations.d.ts /app/declarations.d.ts
# set working directory

# install dependencies needed to compile canvas (needed for Vega-embed)
ENV PYTHONUNBUFFERED=1
RUN apk add --update --no-cache \
    python3 \
    pkgconfig \
    pixman-dev \
    cairo-dev \
    pango-dev \
    alpine-sdk
RUN ln -sf python3 /usr/bin/python
RUN python3 -m ensurepip
RUN pip3 install --no-cache --no-cache-dir  --upgrade pip setuptools

# install the app, including the dev dependencies
RUN npm ci

COPY nx.json  /app/nx.json	
COPY angular.json /app/angular.json
#copy source
COPY libs /app/libs
COPY apps /app/apps

# generate build
# Redifining the env *might* correct cache invalidation issue
ENV APP=${APP}
ENV node_options=--max_old_space_size=6144
RUN nx build ${APP} --prod --with-deps

############
### prod ###
############

# base image
FROM base as prod


WORKDIR /app

#Copy over dependency list
COPY package.json /app/package.json
# install the app and include only dependencies needed to run
RUN npm install --only=production  --legacy-peer-deps
# copy artifact build from the 'build environment'
RUN echo app is ${APP}
COPY --from=build /app/dist/apps/${APP}/ .
EXPOSE 3333
CMD node main.js

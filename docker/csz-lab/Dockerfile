FROM --platform=linux/amd64 node:16.13.2 AS base
LABEL csz.web.image.authors="daniel.boehler@uzh.ch" \
      csz.web.image.version="1.0"

# install dependencies
RUN apt-get update && apt-get install -y \
    git \
    yarn \
    && rm -rf /var/lib/apt/lists/*

RUN adduser csz
RUN mkdir /app && chown -R csz:csz /app
USER csz

WORKDIR /app/csz-lab

# install app dependencies
COPY --chown=csz:csz csz-lab-app/package.json csz-lab-app/package-lock.json* /app/csz-lab/

# clone pybossa into the container (from github)
# RUN git clone --recursive https://github.com/CitizenScienceCenter/csz-lab-app /app/csz-lab

RUN yarnpkg install



FROM base AS build

# alternative: local copy of csz to the container
COPY --chown=csz:csz csz-lab-app /app/csz-lab

ARG ENV_FILE_NAME=prod.env.js
COPY --chown=csz:csz config/env/${ENV_FILE_NAME} /app/csz-lab/config/prod.env.js

RUN yarnpkg run build


FROM nginx:latest as deploy
LABEL ch.citizenscience.env="PROD"
COPY --from=build /app/csz-lab/dist /usr/share/nginx/html
COPY csz-lab-app/public /usr/share/nginx/html/public
#COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


FROM --platform=linux/amd64 node:16.13.2 AS base
LABEL c3s.web.image.authors="daniel.boehler@uzh.ch" \
      c3s.web.image.version="1.0"

# install dependencies
RUN apt-get update && apt-get install -y \
    git \
    yarn \
    && rm -rf /var/lib/apt/lists/*

RUN adduser c3s
RUN mkdir /app && chown -R c3s:c3s /app
USER c3s

WORKDIR /app/c3s-lab

# install app dependencies
COPY --chown=c3s:c3s c3s-lab-client/package.json c3s-lab-client/package-lock.json* /app/c3s-lab/

# clone pybossa into the container (from github)
# RUN git clone --recursive https://github.com/CitizenScienceCenter/c3s-lab-client /app/c3s-lab

RUN yarnpkg install



FROM base AS build

# alternative: local copy of c3s to the container
COPY --chown=c3s:c3s c3s-lab-client /app/c3s-lab

ARG ENV_FILE_NAME=prod.env.js
COPY --chown=c3s:c3s config/env/${ENV_FILE_NAME} /app/c3s-lab/config/prod.env.js

RUN yarnpkg run build


FROM nginx:latest as deploy
LABEL ch.citizenscience.env="PROD"
COPY --from=build /app/c3s-lab/dist /usr/share/nginx/html
#COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


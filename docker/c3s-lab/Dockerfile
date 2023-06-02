FROM --platform=linux/amd64 node:latest AS base
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



FROM base AS build

WORKDIR /app/c3s-lab

# install dependencies
COPY --chown=c3s:c3s c3s-lab-client/package*.json /app/c3s-lab/
RUN yarn install

# clone pybossa into the container (from github)
# RUN git clone --recursive https://github.com/CitizenScienceCenter/c3s-lab-client /app/c3s-lab

# alternative: local copy of c3s to the container
COPY --chown=c3s:c3s c3s-lab-client /app/c3s-lab
COPY --chown=c3s:c3s config/env/test.env.js /app/c3s-lab/config/prod.env.js

RUN yarn run build

# RUN npm install && npm run build

# replace env vars based on different deployments (dev, test, stag, prod) 
# build application
#COPY --chown=c3s:c3s --chmod=0755 docker/c3s-lab-client/entrypoint.sh /app/entrypoint.sh
#ENTRYPOINT ["/app/entrypoint.sh"]



FROM nginx:latest as deploy
LABEL ch.citizenscience.env="PROD"
COPY --from=build /app/c3s-lab/dist /usr/share/nginx/html
#COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]


# Multi-stage Dockerfile for CSZ Lab - Development and Production
FROM --platform=linux/amd64 node:20 AS base

LABEL csz.web.image.authors="daniel.boehler@uzh.ch" \
      csz.web.image.version="1.0"

# Install system dependencies
RUN apt-get update && apt-get install -y  --no-install-recommends\
    git \
    && rm -rf /var/lib/apt/lists/*

# Create user for security
RUN adduser csz
RUN mkdir /app && chown -R csz:csz /app
USER csz

WORKDIR /app/csz-lab

# Copy package files first for better caching
COPY --chown=csz:csz csz-lab-app/package.json ./


# Install dependencies with legacy peer deps to handle old dependencies
RUN npm install --legacy-peer-deps

# Development stage
FROM base AS development

LABEL csz.web.image.version="1.0-dev"

# Copy the entire app source
COPY --chown=csz:csz csz-lab-app .

# Copy development environment config
COPY --chown=csz:csz config/env/dev.env.js ./config/dev.env.js

# Expose the development server port
EXPOSE 8080

# Start the development server with Docker-optimized config
CMD ["npm", "run", "dev:docker", "--"]

# Build stage for production
FROM base AS build

LABEL csz.web.image.version="1.0-build".

# Copy the entire app source
COPY --chown=csz:csz csz-lab-app .

# clone csz-lab into the container (from github)
RUN git clone --recursive https://github.com/CitizenScienceCenter/csz-lab-app .

# Copy environment config (defaults to prod.env.js)
ARG ENV_FILE_NAME=prod.env.js
COPY --chown=csz:csz config/env/${ENV_FILE_NAME} ./config/prod.env.js

# Build the application
RUN npm run build

# Production deployment stage
FROM nginx:latest AS deploy

LABEL ch.citizenscience.env="PROD" \
      csz.web.image.version="1.0-prod"

# Copy built application from build stage
COPY --from=build /app/csz-lab/dist /usr/share/nginx/html
COPY csz-lab-app/public /usr/share/nginx/html/public

# Expose port 80
EXPOSE 80

# Start nginx
CMD ["nginx", "-g", "daemon off;"]
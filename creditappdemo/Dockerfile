# syntax=docker/dockerfile:1

FROM node:22-alpine AS build

WORKDIR /app

COPY package*.json ./

RUN npm ci

COPY index.html vite.config.js ./
COPY src ./src

ARG VITE_SCORING_MODE=mock
ARG VITE_BACKEND_BASE_URL=http://localhost:8080
ARG VITE_SCORING_PATH=/api/v1/credit-scoring/simulate
ARG VITE_SCORING_SCHEMA=aegira
ARG VITE_SCORING_DEFAULT_TENURE=12

ENV VITE_SCORING_MODE=$VITE_SCORING_MODE
ENV VITE_BACKEND_BASE_URL=$VITE_BACKEND_BASE_URL
ENV VITE_SCORING_PATH=$VITE_SCORING_PATH
ENV VITE_SCORING_SCHEMA=$VITE_SCORING_SCHEMA
ENV VITE_SCORING_DEFAULT_TENURE=$VITE_SCORING_DEFAULT_TENURE

RUN npm run build

FROM nginx:1.27-alpine AS runtime

COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY --from=build /app/dist /usr/share/nginx/html

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]

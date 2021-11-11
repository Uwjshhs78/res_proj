# Build stage
FROM node:current AS build-env
WORKDIR /build
COPY . .
RUN npm install && npm run build

FROM nginx:stable
COPY --from=build-env /build/dist /usr/share/nginx/html/
EXPOSE 80
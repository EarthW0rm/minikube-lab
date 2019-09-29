FROM alpine:3.7 AS SystemOS
RUN apk update \
	&& apk add --update nodejs nodejs-npm \
    && npm install pm2 -g \
    && apk add --update bash && rm -rf /var/cache/apk/*

FROM SystemOS as build
COPY ./ /app-build
WORKDIR /app-build
RUN npm i
RUN npm run production

FROM nginx:1.17.4
COPY --from=build './app-build/public' /usr/share/nginx/html

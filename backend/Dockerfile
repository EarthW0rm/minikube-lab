FROM alpine:3.7 AS SystemOS
RUN apk update \
	&& apk add --update nodejs nodejs-npm \
    && npm install pm2 -g \
    && apk add --update bash && rm -rf /var/cache/apk/*

FROM SystemOS
COPY ./ /app-build
WORKDIR /app-build
RUN npm i
CMD pm2-runtime src/loader.js
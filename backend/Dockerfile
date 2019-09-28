FROM node:8.16.1
COPY ./ /app-build
WORKDIR /app-build
RUN npm install pm2 -g
RUN npm i
CMD pm2-runtime src/loader.js
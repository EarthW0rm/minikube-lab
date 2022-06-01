FROM node AS SystemOS
RUN  npm install pm2 -g 

FROM SystemOS as build
COPY ./ /app-build
WORKDIR /app-build
RUN npm i  --legacy-peer-deps
RUN npm run production

FROM nginx:1.17.4
COPY --from=build './app-build/public' /usr/share/nginx/html

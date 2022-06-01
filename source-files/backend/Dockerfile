FROM node AS SystemOS
RUN npm install pm2 -g 

FROM SystemOS
COPY ./ /app-build
WORKDIR /app-build
RUN npm i
CMD pm2-runtime src/loader.js
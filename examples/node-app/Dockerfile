FROM node:0.10.38-slim

ADD package.json /tmp/package.json
RUN cd /tmp && npm install
RUN mkdir -p /opt/app && cp -a /tmp/node_modules /opt/app/

WORKDIR /opt/app

EXPOSE 8080

ADD server.js package.json /opt/app/

ENTRYPOINT node server.js

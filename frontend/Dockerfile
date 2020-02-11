FROM node:13.7.0-alpine3.10
ARG REACT_APP_BACKEND_HOST

WORKDIR /home/node/app
COPY ./src/ ./
RUN npm install

CMD npm start

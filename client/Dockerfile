FROM node:16-alpine

LABEL version="1.0"
LABEL description="React Frontend for the Library API"

WORKDIR /app

COPY ./package.json ./

RUN npm install

COPY . .

CMD [ "npm", "run", "start" ]


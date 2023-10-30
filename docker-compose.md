docker-compose.md

# Use root/example as user/password credentials
version: '3.1'

services:
  watchgodot-api:
    image: watchgodot
    build:
      context: .
      dockerfile: Dockerfile
    restart: always
    volumes:
      - ./:/usr/src/app
      - ../repositories:/usr/src/app/../repositories
    ports:
      - "3063:3000"
    links:
      - mongo

  mongo:
    image: mongo:4
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: example
      MONGO_INITDB_DATABASE: GodotRepos
    volumes:
      - ./dataDB:/data/db
      - ./mongo-init.js:/docker-entrypoint-initdb.d/mongo-init.js:ro

  mongo-express:
    image: mongo-express
    restart: always
    ports:
      - 8061:8081
    environment:
      ME_CONFIG_MONGODB_ADMINUSERNAME: root
      ME_CONFIG_MONGODB_ADMINPASSWORD: example
      ME_CONFIG_MONGODB_URL: mongodb://root:example@mongo:27017/

  theia:
    image: elswork/theia
    restart: always
    ports:
      - 3060:3000
    volumes:
      - .:/home/project:cached

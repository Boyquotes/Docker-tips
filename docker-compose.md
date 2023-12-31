docker-compose.md
```
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
    image: theia-blueprint
    restart: always
    ports:
      - 3060:3000
    volumes:
      - .:/home/project:cached


================
PMA MARIADB https://david.dev/how-to-install-mariadb-phpmyadmin-with-docker-compose
version: '3'

volumes:
  mariadb:
    driver: local

networks:
    db:
        driver: bridge

services:
  mariadb:
    image: mariadb:10.6
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: YOUR_ROOT_PASSWORD_HERE
      MYSQL_USER:  YOUR_MYSQL_USER_HERE 
      MYSQL_PASSWORD: YOUR_USER_PW_HERE
    expose:
        - "40000"
    ports:
        - "40000:3306"
    volumes:
     - mariadb:/var/lib/mysql
    networks:
      db:
              
  phpmyadmin:
    image: phpmyadmin
    restart: always
    expose:
      - "40001"
    ports:
      - "40001:80"
    environment:
      - PMA_HOST=mariadb
      - PMA_PORT=3306 
    networks:
      db:
```
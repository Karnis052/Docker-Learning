# Docker-Learning

## <a name="code-snippets">🕸️ Code Snippets</a>
<details>
<summary><code>vite-docker/compose.yaml</code></summary>

```yaml
# Comments are provided throughout this file to help you get started.
# If you need more help, visit the Docker Compose reference guide at
# https://docs.docker.com/go/compose-spec-reference/

# Here the instructions define your application as a service called "server".
# This service is built from the Dockerfile in the current directory.
# You can add other services your application may depend on here, such as a
# database or a cache. For examples, see the Awesome Compose repository:
# https://github.com/docker/awesome-compose
services:
  web:
    build:
      context: .
    ports:
      - 5173:5173
    volumes:
      - .:/app 
      - /app/node_modules
    environment:
      - CHOKIDAR_USEPOLLING=true 
      - WATCHPACK_POLLING=true


# The commented out section below is an example of how to define a PostgreSQL
# database that your application can use. `depends_on` tells Docker Compose to
# start the database before your application. The `db-data` volume persists the
# database data between container restarts. The `db-password` secret is used
# to set the database password. You must create `db/password.txt` and add
# a password of your choosing to it before running `docker-compose up`.
#     depends_on:
#       db:
#         condition: service_healthy
#   db:
#     image: postgres
#     restart: always
#     user: postgres
#     secrets:
#       - db-password
#     volumes:
#       - db-data:/var/lib/postgresql/data
#     environment:
#       - POSTGRES_DB=example
#       - POSTGRES_PASSWORD_FILE=/run/secrets/db-password
#     expose:
#       - 5432
#     healthcheck:
#       test: [ "CMD", "pg_isready" ]
#       interval: 10s
#       timeout: 5s
#       retries: 5
# volumes:
#   db-data:
# secrets:
#   db-password:
#     file: db/password.txt


```

</details>

<details>
<summary><code>mern-docker/frontend/Dockerfile</code></summary>

```dockerfile
FROM node:20 
WORKDIR /app 
COPY package*.json ./
RUN npm install 
COPY  . . 
EXPOSE 5173 
CMD npm run dev 

```

</details>

<details>
<summary><code>mern-docker/backend/Dockerfile</code></summary>

```dockerfile
FROM node:20 
WORKDIR /app 
COPY package*.json ./
RUN npm install 
COPY  . . 
EXPOSE 8000
CMD npm start

```

</details>

<details>
<summary><code>mern-docker/compose.yaml</code></summary>

```yaml
# specify the version of docker-compose
version: "3.8"
services:
 
  web:
    depends_on: 
      - api
    build: ./frontend
    image: karnis052/mern-docker-web:latest
    ports:
      - 5173:5173
    environment:
      VITE_API_URL: http://localhost:8000
    develop:
      watch:
        - path: ./frontend/package.json
          action: rebuild

        - path: ./frontend/package-lock.json
          action: rebuild
          
        - path: ./frontend
          target: /app
          action: sync
  api: 

    depends_on: 
      - db
    build: ./backend
    image: karnis052/mern-docker-api:latest
    ports: 
      - 8000:8000
    environment: 
      DB_URL: mongodb://db/anime
    develop:
      watch:
        - path: ./backend/package.json
          action: rebuild
        - path: ./backend/package-lock.json
          action: rebuild
      
        - path: ./backend
          target: /app
          action: sync

  db:
    image: mongo:latest
    ports:
      - 27017:27017
    volumes:
      - anime:/data/db

volumes:
  anime:
```

</details>


# Docker-Learning

## <a name="code-snippets">🕸️ Code Snippets</a>
<details>
<summary><code>vite-docker/compose.yaml</code></summary>

```yaml
# define the services/containers to be run
services:
  # define the application container/service
  # we can use any name for the service. Here it is `web`
  # we can create multiple services as well
  web:
    # specify the image to build the container from
    # this can be any image available in docker hub or a custom one or the one we want to build
    build:
      # specify the path to the Dockerfile
      context: .
      # specify the file name (optional)
      dockerfile: Dockerfile
    
    # specify the port mapping from host to the container
    # this is similar to the -p flag in `docker run` command
    # first port is the port on host machine and the second is the port inside the container
    ports:
      - 5173:5173

    # specify the volumes to mount
    # what this does is it mounts the current directory to the `/app` directory inside the container. 
    # due to this, any changes made to the files in the current directory will be reflected inside the container. It is similar to the -v flag in `docker run` command. 
    # even if a container is stopped or deleted, volumes are not deleted and can be used by other containers as well.
    volumes:
      # over here, we are mounting the current directory to the `/app` directory inside the container (which is the working directory of the container)
      # syntax is `<path to the directory on host>:<path to the directory inside the container>`
      # we're doing this because we want to reflect the changes made to the files in the current directory inside the container
      - .:/app
      # we also mount the node_modules directory inside the container at /app/node_modules. This is done to avoid installing the node_modules inside the container. 
      # node_modules will be installed on the host machine and mounted inside the container
      - /app/node_modules
```

</details>

<details>
<summary><code>mern-docker/frontend/Dockerfile</code></summary>

```dockerfile
FROM node:20-alpine3.18

# RUN addgroup app && adduser -S -G app app

# USER app

WORKDIR /app

COPY package*.json ./

# USER root

# RUN chown -R app:app .

# USER app

RUN npm install

COPY . .

EXPOSE 5173

CMD npm run dev
```

</details>

<details>
<summary><code>mern-docker/backend/Dockerfile</code></summary>

```dockerfile
FROM node:20-alpine3.18

RUN addgroup app && adduser -S -G app app

USER app

WORKDIR /app

COPY package*.json ./

# change ownership of the /app directory to the app user
USER root

# change ownership of the /app directory to the app user
# chown -R <user>:<group> <directory>
# chown command changes the user and/or group ownership of for given file.
RUN chown -R app:app .

# change the user back to the app user
USER app

RUN npm install

COPY . . 

EXPOSE 8000 

CMD npm start
```

</details>

<details>
<summary><code>mern-docker/compose.yaml</code></summary>

```yaml
# specify the version of docker-compose
version: "3.8"

# define the services/containers to be run
services:
  # define the frontend service
  # we can use any name for the service. A standard naming convention is to use "web" for the frontend
  web:
    # we use depends_on to specify that service depends on another service
    # in this case, we specify that the web depends on the api service
    # this means that the api service will be started before the web service
    depends_on: 
      - api
    # specify the build context for the web service
    # this is the directory where the Dockerfile for the web service is located
    build: ./frontend
    # specify the ports to expose for the web service
    # the first number is the port on the host machine
    # the second number is the port inside the container
    ports:
      - 5173:5173
    # specify the environment variables for the web service
    # these environment variables will be available inside the container
    environment:
      VITE_API_URL: http://localhost:8000

    # this is for docker compose watch mode
    # anything mentioned under develop will be watched for changes by docker compose watch and it will perform the action mentioned
    develop:
      # we specify the files to watch for changes
      watch:
        # it'll watch for changes in package.json and package-lock.json and rebuild the container if there are any changes
        - path: ./frontend/package.json
          action: rebuild
        - path: ./frontend/package-lock.json
          action: rebuild
        # it'll watch for changes in the frontend directory and sync the changes with the container real time
        - path: ./frontend
          target: /app
          action: sync

  # define the api service/container
  api: 
    # api service depends on the db service so the db service will be started before the api service
    depends_on: 
      - db

    # specify the build context for the api service
    build: ./backend
    
    # specify the ports to expose for the api service
    # the first number is the port on the host machine
    # the second number is the port inside the container
    ports: 
      - 8000:8000

    # specify environment variables for the api service
    # for demo purposes, we're using a local mongodb instance
    environment: 
      DB_URL: mongodb://db/anime
    
    # establish docker compose watch mode for the api service
    develop:
      # specify the files to watch for changes
      watch:
        # it'll watch for changes in package.json and package-lock.json and rebuild the container and image if there are any changes
        - path: ./backend/package.json
          action: rebuild
        - path: ./backend/package-lock.json
          action: rebuild
        
        # it'll watch for changes in the backend directory and sync the changes with the container real time
        - path: ./backend
          target: /app
          action: sync

  # define the db service
  db:
    # specify the image to use for the db service from docker hub. If we have a custom image, we can specify that in this format
    # In the above two services, we're using the build context to build the image for the service from the Dockerfile so we specify the image as "build: ./frontend" or "build: ./backend".
    # but for the db service, we're using the image from docker hub so we specify the image as "image: mongo:latest"
    # you can find the image name and tag for mongodb from docker hub here: https://hub.docker.com/_/mongo
    image: mongo:latest

    # specify the ports to expose for the db service
    # generally, we do this in api service using mongodb atlas. But for demo purposes, we're using a local mongodb instance
    # usually, mongodb runs on port 27017. So we're exposing the port 27017 on the host machine and mapping it to the port 27017 inside the container
    ports:
      - 27017:27017

    # specify the volumes to mount for the db service
    # we're mounting the volume named "anime" inside the container at /data/db directory
    # this is done so that the data inside the mongodb container is persisted even if the container is stopped
    volumes:
      - anime:/data/db

# define the volumes to be used by the services
volumes:
  anime:
```

</details>

<details>
<summary><code>next-docker/Dockerfile</code></summary>

```dockerfile
# inherit from a existing image to add the functionality
FROM node:20-alpine3.18

# RUN addgroup app && adduser -S -G app app
# USER app

# Set the working directory and assign ownership to the non-root user
WORKDIR /app

# Copy the package.json and package-lock.json files into the image.
COPY package*.json ./

# change ownership of the /app directory to the app user
# USER root

# change ownership of the /app directory to the app user
# chown -R <user>:<group> <directory>
# chown command changes the user and/or group ownership of for given file.
# RUN chown -R app:app .

# change the user back to the app user
# USER app

# Install the dependencies.
RUN npm install

# Copy the rest of the source files into the image.
COPY . .

# Expose the port that the application listens on.
EXPOSE 3000

# Run the application.
CMD npm run dev
```

</details>

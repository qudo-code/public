
# Docker
![](https://raw.githubusercontent.com/qudo-code/public/refs/heads/main/media/Pasted%20image%2020251202165028.png)
Docker at its core is a tool for organizing virtual machines. In this guide I go over how I use it for developing and hosting all my projects.
## Concepts

### Virtual Machines
Virtual machines are computer environments that run inside another computer. For example, on my Mac, I could setup a Linux VM and run Linux in a separate window. On that VM I could install stuff as if it was another laptop and my Mac isn't affected.

Docker is a way to create VMs based on a configs. An example of a Docker VM could be one that contains Linux + Postgres already installed. You could download the image, run it, and have a database running in a VM on localhost without setting up Postgres on your main machine.

### Images vs. Containers
#### Images
Images are read only virtual machine templates. Imagine you have a script you use to setup your computer and install required apps whenever you buy a new one. Images are sort of like that setup script. For example, if we use the `postgres` image, that contains a Linux computer with Postgres already installed for us to use.

#### Containers
Containers are instances/VMs of images. The image contains instructions on how to setup the VM, the container is the running VM you created from that image.

### Hosting
For hosting you'll need to find a platform that supports building and running Docker containers. I like [Railway](https://railway.com/templates) or [DigitalOcean Droplets](https://www.digitalocean.com/products/droplets).

## Install
### EZ Mode
Easiest way to install Docker is by installing "Docker Desktop". This will come with a pretty UI for managing your containers and all things you need in one install.

https://docs.docker.com/docker-hub/quickstart/

### Homebrew
If on Mac and don't need the UI, you can get the Docker CLI from Homebrew.

```bash
brew install docker
```

### NixOS
For NixOS I add the `docker` package, enable virtualization, grant permissions to user.
```nix
environment.systemPackages = with pkgs; [
    docker
];

virtualisation.docker.enable = true;
users.extraGroups.docker.members = [ "myuser" ];
```

## Manage Containers
Before we create our own containers, let's make sure we know how to view what containers are already running. If using the CLI, run `docker ps`.
### Docker Desktop

If you installed "Docker Desktop", you should be able to search for "Docker" as an app on your computer and start it. On Mac, once running you'll see a little boat icon in the status bar which you can click and select "Go to the Dashboard".

### VSCode Extension

VSCode based editors have extensions you can get to view your containers right in IDE such as this official one by Docker. These extentions are also kinda cool because you can do stuff like double click a container and it will open a new terminal session connected to that container right in VSCode.

https://code.visualstudio.com/docs/containers/overview

### CLI Cheatsheet
```bash
# Manage Containers
# ---------------------------------------------------
docker ps
docker start CONTAINER_ID
docker stop CONTAINER_ID
docker rm -F CONTAINER_ID

# Compose
# ---------------------------------------------------
# Create and start services defined in ./docker-compose.yml
docker compose up -d
docker compose up -d api # api only
docker compose down --remove-orphans --volumes
docker compose down api --remove-orphans --volumes # api only

# Build
# ---------------------------------------------------
# Build ./Dockerfile as image named "dockertut-app"
# "." means we are using the current directory as context
docker build -f ./Dockerfile -t dockertut-app .

# Terminal/Exec
# ---------------------------------------------------
# Run command on container
docker exec -it CONTAINER_ID echo "Hello world"
# Attach terminal to container
docker attach CONTAINER_ID
```

## Docker Compose
Docker compose allows us to define how to start services with a `docker-compose.yml` file.

Example `docker-compose.yml` that creates a Postgres database.

```yml
name: dockertut
networks:
  dockertut-private-network:
    driver: bridge
services:
  devdb:
    container_name: dockertut-db
    image: postgres
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: dockertut
    networks:
      - dockertut-private-network
```
- `name: dockertut` at the top defines a group name for all of our services to go under.
- `networks:` a few networks are defined at the top. Usually I'll set DB and API to run on private network, then use public for web/UI service.
- `devdb` under services is an alias to that particular service config and what we would use to start/stop a specific service in our compose file.
- `container_name: dockertut-db` defines what our database container will be called once running.
- `image: postgres` is a public image from Docker Hub we can use.

See [Docker Hub](https://hub.docker.com/) for more images.

### Compose CLI
```bash
docker compose up -d # start all services in local compose file
docker compose up -d api # api only
docker compose down --remove-orphans --volumes # delete all services in local compose file
docker compose down api --remove-orphans --volumes # api only
```

## Docker Build
So far we have learned how to create a container from an existing Postgres image. Now let's learn how to build our own images using a `Dockerfile`.

You'll need to be familiar with `Dockerfile`s if you plan to host your app with Docker. A `Dockerfile` will specify things like the OS, environment variables, network settings, app build/start commands, etc...

Example `Dockerfile`
```dockerfile
# Start with image from Bun.
# This wil be a Linux machine with Bun already installed.
FROM oven/bun:canary

# Create a directory and navigate to it.
# Basically "mkdir /app && cd /app".
WORKDIR /app

# Copy project files to current directory on image.
# Since we ran the WORKDIR above, we are copying to /app
COPY . .

# Update and add system dependencies
RUN apt-get update && apt-get install unzip

# Install app dependencies
RUN bun install

# Define container port
EXPOSE 3000

# What to run when then image is used (start command)
CMD bun start:api
```

### Build/Start w/CLI
Build `./Dockerfile` as image named "dockertut-app". The "." means we are using the current directory as context.
```bash
docker build -f ./Dockerfile -t dockertut-app .
docker ps
docker start CONTAINER_ID
```

### Build w/Compose File
You can also specify a `Dockerfile` to build in `docker-compose.yml` files, then use the `docker up/down` CLI mentioned above.
```yml
  api:
    container_name: soldir-api
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
```

## Example Project
A simple [Bun](https://bun.sh/) web server that uses Docker for development and hosting.

```bash
curl -fsSL https://bun.sh/install | bash
```
### Initialize
Create a new folder for our project with a `package.json` like the one below.
```bash
mkdir ~/docker-example
cd ~/docker-example
```

Navigate to your empty project folder and create the following files.

`./package.json`
```json
{
  "name": "docker-example",
  "module": "index.ts",
  "type": "module",
  "script": {
    "dev": "bun run --watch index.ts",
    "start": "bun index.ts",
    "dup": "npm docker:down && bun docker:up",
    "docker:up": "docker compose up -d && sleep 2 && npm db:push",
    "docker:down": "docker compose down --remove-orphans --volumes",
    "docker:stop": "docker compose stop",
    "docker:start": "docker compose start",
    "docker:build": "docker build -f ./Dockerfile.ui -t dockertut-app .",
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
  },
  "devDependencies": {
    "@types/bun": "latest",
		"prisma": "^6.4.1"
  },
  "peerDependencies": {
    "typescript": "^5.0.0",
  },
  "dependencies": {
    "@prisma/client": "^6.4.1"
  }
}
```
`./tsconfig.json`
```json
{
  "compilerOptions": {
    // Enable latest features
    "lib": ["ESNext", "DOM"],
    "target": "ESNext",
    "module": "ESNext",
    "moduleDetection": "force",
    "jsx": "react-jsx",
    "allowJs": true,

    // Bundler mode
    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "verbatimModuleSyntax": true,
    "noEmit": true,

    // Best practices
    "strict": true,
    "skipLibCheck": true,
    "noFallthroughCasesInSwitch": true,

    // Some stricter flags (disabled by default)
    "noUnusedLocals": false,
    "noUnusedParameters": false,
    "noPropertyAccessFromIndexSignature": false
  }
}
```
`./index.ts`
```ts
const server = Bun.serve({
  port: 3000,
  fetch(request) {
    return new Response("Welcome to Bun!");
  },
});

console.log(`Listening on ${server.url}`);
````
`./docker-compose.yml`
```yml
name: dockertut
networks:
  dockertut-private-network:
    driver: bridge

services:
  devdb:
    container_name: dockertut-db
    image: postgres
    restart: always
    ports:
      - 5432:5432
    environment:
      POSTGRES_USER: root
      POSTGRES_PASSWORD: mysecretpassword
      POSTGRES_DB: dockertut
    networks:
      - dockertut-private-network
```
`./Dockerfile`
```Dockerfile
FROM oven/bun:canary

WORKDIR /app
COPY . .

RUN apt-get update && apt-get install unzip

RUN bun install
EXPOSE 3000

CMD bun start:api
```

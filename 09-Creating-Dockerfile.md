# Creating and Using a Dockerfile to Containerize Node.js Applications

## Introduction to Dockerfiles

In the previous sections, you learned about Docker containers and images, as well as CLI commands. Now it's time to create custom Docker images using a Dockerfile.

The goal is to solve the original problem: take your application code and create a Docker image from it so that developers, production environments, and staging environments all can run the same containerized application.

## The Problem We're Solving

Remember the original issue where:

- You needed to replicate your development environment across multiple developers
- Each developer had different OS versions (MacBook, Windows, Linux)
- Setting up all dependencies was time-consuming and error-prone
- Production environments often had inconsistencies

With Docker, you can:

- Take your entire project
- Create a Docker image containing your application
- Share this image so others can run it in containers
- Ensure consistency across all environments (dev, staging, production)

## Example Application: Node.js Express Server

Let's work with a simple Node.js application:

### Project Structure

```
project/
├── package.json
├── index.js
└── Dockerfile
```

### package.json

```json
{
  "name": "docker-app",
  "version": "1.0.0",
  "scripts": {
    "start": "node index.js"
  },
  "dependencies": {
    "express": "^4.x.x"
  }
}
```

### index.js

```javascript
const express = require("express");
const app = express();

const port = process.env.PORT || 8000;

app.get("/", (req, res) => {
  res.json({
    status: "success",
    message: "Hello from the Express server",
  });
});

app.listen(port, () => {
  console.log(`Server is running on port ${port}`);
});
```

This is a simple Express.js application that:

- Listens on port 8000
- Returns a JSON response on the root route
- Has a basic HTTP server

## Running the Application Locally

To run this application without Docker, you need:

1. **Node.js installed** (specific version, e.g., version 18)
2. **npm installed** (specific version, e.g., version 10)
3. **Node_modules installed** (via npm install)

### Requirements List

To ensure the application works correctly, you would document:

- Node.js version 18
- npm version 10
- Express package and dependencies

## Creating a Dockerfile

A Dockerfile contains instructions to build a Docker image. The basic syntax is:

```dockerfile
FROM [base_image]
RUN [command]
COPY [source] [destination]
WORKDIR [directory]
ENTRYPOINT [command]
```

### Sample Dockerfile

```dockerfile
# Base Image
FROM ubuntu

RUN apt-get update
RUN apt install -y curl
RUN curl -sL https://deb.nodesource.com/setup_18.x -o /tmp/nodesource_setup.sh
RUN bash /tmp/nodesource_setup.sh
RUN apt install -y nodejs

# Copying the source code to docker image
COPY index.js /home/app/index.js
COPY package-lock. json /home/app/package-lock.json
COPY package.json /home/app/package.json

WORKDIR /home/app/

RUN npm install
```

### Understanding Each Line

#### FROM ubuntu

- Sets the base image
- This is the foundation operating system for your container
- All subsequent commands run on top of this

#### RUN apt-get update

- Updates the package manager's package list
- Prepares the system for installing software

#### RUN apt install -y curl

- Installs curl
- The `-y` flag automatically confirms installation

#### RUN curl -sL https://deb.nodesource.com/setup_18.x -o /tmp/nodesource_setup.sh

- Downloads the Node.js setup script from NodeSource repository
- Executes it to add Node.js 18 to available packages
- This is the proper way to install Node.js on Ubuntu

#### RUN apt install -y nodejs

- Installs Node.js on the system
- npm comes automatically with Node.js

#### COPY package.json /home/app/

- Copies package.json from your local machine to the container
- Source: Your local package.json
- Destination: /home/app/ inside the container

#### COPY index.js /home/app/

- Copies the application source code to the container
- Ensures your code is available inside the image

#### WORKDIR /home/app

- Sets the working directory for subsequent commands
- All future commands execute from this directory

#### RUN npm install

- Installs all dependencies listed in package.json
- Generates node_modules folder
- Creates the package-lock.json file

## Building the Docker Image

### Build Command

```bash
docker build -t my_app .
```

**Parameters**:

- `docker build`: The build command
- `-t my_app`: Tags (names) your image as "my_app"
- `.`: The context path (current directory) where Dockerfile is located

### Build Process

When you run the build command, Docker:

1. **Reads the Dockerfile** from the specified context directory
2. **Executes each layer** (each RUN, COPY, WORKDIR command)
3. **Caches layers** for optimization on subsequent builds
4. **Creates the image** after all steps complete

### Layer System

Each command in the Dockerfile creates a layer:

1. `FROM ubuntu` → Layer 1
2. `RUN apt-get update` → Layer 2
3. `RUN apt install...` → Layer 3
4. (and so on...)

**Important**: The order of layers matters. More on this later.

## Docker Build Cache

Docker caches layers for efficiency:

- If you build again without changes, Docker reuses cached layers
- If you modify a layer, Docker rebuilds from that point onward
- All layers below the modified one are also rebuilt

**Example**: If you change your index.js, only the COPY and subsequent layers rebuild. Layers for Ubuntu, Node.js installation are cached.

## Docker Image Size

After building, your image size depends on:

- Base image size (Ubuntu is ~77MB)
- Installed packages
- Your application code
- Node_modules

For a simple Express app, you might end up with an image around 367+ MB.

**Note**: This size will be discussed in optimization techniques in later sections.

## Verifying the Build

After successful build:

```bash
docker images
```

Your new image "my_app" should appear in the list with its size.

## Running Your Custom Image

After building, you can run a container from your custom image:

```bash
docker run -it my_app
```

This:

- Creates a container from your my_app image
- Runs the ENTRYPOINT command (npm start)
- Starts your Express server
- The server runs on port 8000 inside the container

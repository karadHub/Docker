# Docker Essentials for Beginners

This guide covers the fundamental concepts of Docker to get you started.

## Key Docker Concepts

### Images
A Docker image is a lightweight, standalone, executable package of software that includes everything needed to run an application: code, runtime, system tools, system libraries, and settings. Images are created from a `Dockerfile`. They are immutable, meaning once an image is created, it cannot be changed. If you need to make changes, you create a new image.

### Containers
A container is a runnable instance of an image. You can create, start, stop, move, or delete a container using the Docker API or CLI. A container is an isolated environment, and it runs on the host machine's kernel. It's like a lightweight virtual machine, but it shares the host OS kernel.

### Dockerfiles
A `Dockerfile` is a text document that contains all the commands a user could call on the command line to assemble an image. `docker build` uses this file to build a new image automatically. It's the blueprint for your container.

---

## Writing a Dockerfile and Building a Container Image

Here is a simple example of a `Dockerfile`:

```dockerfile
# Use an official lightweight Alpine Linux image as a parent image
FROM alpine:latest

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Run a command when the container launches
CMD ["echo", "Hello from inside the container!"]
```

To build an image from this `Dockerfile`, save it as `Dockerfile` in a directory, and run the following command from that directory:

```bash
docker build -t my-first-image .
```

- `-t my-first-image`: This tags your image with a name (`my-first-image`).
- `.`: This tells Docker to look for the `Dockerfile` in the current directory.

---

## Managing Containers

Once you have an image, you can run it as a container.

### Running a Container
To run a container from the `my-first-image` we just built:

```bash
docker run my-first-image
```
You should see the output: `Hello from inside the container!`

To run a container in the background (detached mode) and map a port:
```bash
# Example with a web server image like nginx
docker run -d -p 8080:80 --name my-web-server nginx
```
- `-d`: Detached mode.
- `-p 8080:80`: Maps port 8080 on the host to port 80 in the container.
- `--name my-web-server`: Gives the container a custom name.

### Listing Containers
To see all running containers:
```bash
docker ps
```
To see all containers (running and stopped):
```bash
docker ps -a
```

### Stopping a Container
To stop a running container, you can use its name or ID:
```bash
docker stop my-web-server
```

### Removing a Container
Once a container is stopped, you can remove it:
```bash
docker rm my-web-server
```
You can force remove a running container with `-f`: `docker rm -f my-web-server`.

---

## Containerizing a Simple Node.js App

Let's containerize a basic "Hello World" Node.js web application.

**1. Create your Node.js app files:**

`package.json`:
```json
{
  "name": "simple-express-app",
  "version": "1.0.0",
  "description": "A simple Node.js app",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.17.1"
  }
}
```

`server.js`:
```javascript
const express = require('express');

const app = express();
const PORT = 8080;

app.get('/', (req, res) => {
  res.send('Hello from the Node.js app inside a Docker container!');
});

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});
```

**2. Create a `Dockerfile`:**

```dockerfile
# Use an official Node.js runtime as a parent image
FROM node:18-alpine

# Set the working directory in the container
WORKDIR /usr/src/app

# Copy package.json and package-lock.json to the working directory
COPY package*.json ./

# Install any needed packages
RUN npm install

# Copy the rest of the application code
COPY . .

# Make port 8080 available to the world outside this container
EXPOSE 8080

# Define the command to run the app
CMD [ "npm", "start" ]
```

**3. Build and run the container:**

```bash
# Build the image
docker build -t my-node-app .

# Run the container
docker run -p 4000:8080 -d --name my-running-app my-node-app
```

Now, you can access your app at `http://localhost:4000` in your browser.

---

## Docker Multi-stage Build

Multi-stage builds are a feature that lets you use multiple `FROM` statements in your `Dockerfile`. Each `FROM` instruction can use a different base, and each one begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image.

This is useful for creating smaller, more secure production images. For example, you can use a larger image with build tools to compile your code, and then copy only the compiled artifacts into a smaller, "slimmer" production image that doesn't contain the build dependencies.

### Example: Multi-stage build for a Node.js app

Let's optimize our previous Node.js app `Dockerfile`.

```dockerfile
# ---- Build Stage ----
# Use a Node.js image for building the app
FROM node:18-alpine AS build

# Set the working directory
WORKDIR /usr/src/app

# Copy package files and install dependencies
COPY package*.json ./
RUN npm install

# Copy the rest of the application code
COPY . .

# ---- Production Stage ----
# Use a smaller, more secure base image for production
FROM node:18-alpine

# Set the working directory
WORKDIR /usr/src/app

# Copy only the necessary files from the build stage
# This includes node_modules and the source code
COPY --from=build /usr/src/app .

# Expose the port the app runs on
EXPOSE 8080

# Command to run the application
CMD [ "npm", "start" ]
```

In this example:
1.  The `build` stage uses `node:18-alpine` to install all dependencies, including `devDependencies`.
2.  The final (production) stage also starts from `node:18-alpine`.
3.  We use `COPY --from=build /usr/src/app .` to copy the application code and the `node_modules` folder from the `build` stage to the final stage.
4.  The final image will be much smaller because it doesn't contain any of the build tools or intermediate files from the first stage.

To build it, the command is the same:
```bash
docker build -t my-node-app-multistage .
```
The resulting `my-node-app-multistage` image will be smaller and more optimized for production than the single-stage build image.
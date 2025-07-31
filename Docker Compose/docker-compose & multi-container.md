# Docker Compose & Multi-Container Apps: The Easy Guide

## Intro to Docker Compose

**Docker Compose** is a tool that lets you define and run multi-container Docker applications. Instead of starting each container with a long `docker run` command, you describe your whole app (all its services) in a single `docker-compose.yml` file. Then, with one command, you can start everything.

**Why use Docker Compose?**
- Real-world apps often need more than one service (like a frontend, backend, and database).
- Compose makes it easy to manage, configure, and network these services together.

## Dockerizing a Multi-Container App

Imagine you have:
- A **frontend** (React app)
- A **backend** (Node.js API)
- A **database** (MongoDB)

Each of these runs in its own container. With Docker Compose, you describe all three in one file.

**Example `docker-compose.yml`:**
```yaml
version: '3'
services:
  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
  backend:
    build: ./backend
    ports:
      - "5000:5000"
    depends_on:
      - db
  db:
    image: mongo:latest
    ports:
      - "27017:27017"
```

- `build: ./frontend` tells Docker to build the frontend image from the `frontend` folder.
- `depends_on` means the backend waits for the database to be ready.
- Each service can have its own ports, environment variables, and volumes.

## Hands-on: Setting Up a Multi-Service App

1. **Create folders** for each service (`frontend`, `backend`).
2. Add a `Dockerfile` in each folder to define how to build that service.
3. Write a `docker-compose.yml` in your project root (see above).
4. Start everything with:
    ```bash
    docker-compose up
    ```
5. All services start together, networked and ready to talk to each other.

**Summary:**  
Docker Compose makes it super easy to run and manage apps with multiple containers. Just describe your stack in YAML, and you're ready for real-world development
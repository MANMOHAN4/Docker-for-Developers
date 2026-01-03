# Introduction to Docker Compose

## The Problem with Multiple Containers

When developing a real application, you typically need multiple services running:

- **Application**: Node.js/TypeScript application
- **Database**: PostgreSQL
- **Cache**: Redis
- **Message queue**: RabbitMQ
- **Other services**: Elasticsearch, etc.

Managing all these containers individually is cumbersome:

```bash
docker run -d --name postgres postgres:15
docker run -d --name redis redis:7
docker run -d --name app my_app_image
# ... and so on
```

Also, sharing your development environment becomes difficult. You'd have to tell teammates:
"Install Redis version 7, PostgreSQL version 15, and ensure they're running..."

## The Solution: Docker Compose

**Docker Compose** is a tool that lets you define and run multiple containers as a single application.

With Docker Compose, one command starts your entire infrastructure:

```bash
docker compose up
```

All services start together, networking is configured, and your development environment is ready.

## Docker Compose File Format

Create a file named `docker-compose.yml` in your project root:

```yaml
name: e-commerce

services:
  db:
    image: postgres:16
    container_name: postgres_container
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: postgres
    ports:
      - "5432:5432"

  redis:
    image: redis:7
    container_name: redis_container
    ports:
      - "6379:6379"

  app:
    image: my_app_image
    container_name: app_container
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
```

## Understanding the Compose File

### Services

```yaml
services:
  db:
    image: postgres:16
```

Defines each service. Each service is a container.

### Service Configuration

- **image**: The Docker image to use
- **container_name**: Optional friendly name for the container
- **environment**: Environment variables to pass to the container
- **ports**: Port mappings (host_port:container_port)
- **depends_on**: Services that must start before this one

## Running Docker Compose

### Starting All Services

```bash
docker compose up
```

This:

1. Creates a network for all services
2. Starts all containers
3. Streams logs to your terminal
4. Keeps containers running

### Running in Background

```bash
docker compose up -d
```

The `-d` flag runs in detached mode, returning control to your terminal.

### Stopping Services

```bash
docker compose down
```

Stops and removes all containers defined in the compose file.

## Practical Example: Full-Stack Application

```yaml
name: e-commerce

services:
  postgres:
    image: postgres:16
    container_name: ecommerce_db
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_USER: postgres
      POSTGRES_DB: ecommerce
    ports:
      - "2:5432"

  redis:
    image: redis:7
    container_name: ecommerce_redis
    ports:
      - "6379:6379"

  app:
    image: ecommerce_app:latest
    container_name: ecommerce_app
    environment:
      DATABASE_URL: postgres://postgres:postgres@postgres:5432/ecommerce
      REDIS_URL: redis://redis:6379
      PORT: 8000
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis
```

## How Services Communicate

Services in the same compose file automatically get DNS names:

- `postgres` service is accessible as `postgres:5432`
- `redis` service is accessible as `redis:6379`

Your application connects using:

```
DATABASE_URL: postgres://postgres:postgres@postgres:5432/ecommerce
REDIS_URL: redis://redis:6379
```

## Common Docker Compose Commands

```bash
docker compose up              # Start all services
docker compose up -d           # Start in background
docker compose down            # Stop and remove containers
docker compose logs            # View logs
docker compose logs app        # View specific service logs
docker compose ps              # List services
docker compose restart         # Restart services
docker compose build           # Build custom images
```

## Benefits of Docker Compose

1. **Single command setup**: `docker compose up` starts everything
2. **Environment reproducibility**: Everyone gets the same setup
3. **Easy networking**: Services communicate by name automatically
4. **Clean teardown**: `docker compose down` cleans everything
5. **Version control**: Compose file is part of your repository
6. **Documentation**: Compose file documents your infrastructure

## For Development

Docker Compose is perfect for development because:

- No need to install PostgreSQL, Redis, etc. locally
- Same environment for all team members
- Easy to add/remove services
- Can quickly spin up/tear down for testing
- Matches production setup (when using same images)

## Key-takeaways

Docker Compose simplifies multi-container development:

1. Define all services in `docker-compose.yml`
2. Run `docker compose up` to start everything
3. Services communicate using service names
4. Easy to modify and extend
5. Perfect for local development and team collaboration

Docker Compose is essential for any non-trivial application with multiple services.

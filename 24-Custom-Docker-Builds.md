# Custom Docker Builds in Docker Compose

## Building Custom Images with Docker Compose

Instead of relying only on pre-built images from Docker Hub, you can build custom Docker images directly from Docker Compose.

## Two Approaches

### Approach 1: Pre-built and Pushed (Less Ideal for Development)

```yaml
services:
  backend:
    image: username/backend-app:latest
    ports:
      - "8000:8000"
```

**Problem**: Every code change requires:

1. Build the image locally
2. Push to Docker Hub
3. Pull in Docker Compose

This is overkill for development.

### Approach 2: Build Within Compose (Recommended for Development)

```yaml
services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
```

## Build Configuration

### Basic Build

```yaml
services:
  app:
    build: .
```

Assumes `Dockerfile` in current directory.

### Specifying Dockerfile Path

```yaml
services:
  app:
    build:
      context: ./app
      dockerfile: Dockerfile.prod
```

- `context`: Directory containing the Dockerfile and source code
- `dockerfile`: Path to the Dockerfile (relative to context)

### Build Arguments

```yaml
services:
  app:
    build:
      context: .
      args:
        BUILD_ENV: production
        VERSION: 1.0.0
```

In Dockerfile:

```dockerfile
ARG BUILD_ENV
ARG VERSION

RUN echo "Building for ${BUILD_ENV} version ${VERSION}"
```

## Using Build with Other Services

```yaml
version: "3.8"

services:
  backend:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis
    environment:
      DATABASE_URL: postgres://db:5432/myapp
      REDIS_URL: redis://redis:6379

  db:
    image: postgres:15

  redis:
    image: redis:7
```

### Network Communication

Important: When using `localhost` inside your application code:

**WRONG** (won't work):

```
postgres://localhost:5432  # localhost refers to the container itself
redis://localhost:6379     # not the redis service
```

**CORRECT** (uses service name):

```
postgres://db:5432         # db is the service name
redis://redis:6379         # redis is the service name
```

## Building and Running

### Build and Start

```bash
docker compose up
```

This automatically builds your image if not already built.

### Rebuild on Changes

```bash
docker compose up --build
```

Rebuilds the image even if it already exists (when you make code changes).

### Build Only

```bash
docker compose build
```

Just build the image without starting containers.

## Exposing Ports Correctly

Only expose the ports you need to access from outside:

```yaml
services:
  backend:
    build: .
    ports:
      - "8000:8000" # Exposed for external access

  database:
    image: postgres:15
    # NO PORT MAPPING - only backend needs external access

  cache:
    image: redis:7
    # NO PORT MAPPING - internal use only
```

## Complete Practical Example

```yaml
version: "3.8"

services:
  backend:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: backend
    ports:
      - "8000:8000"
    depends_on:
      - postgres
      - redis
    environment:
      DATABASE_URL: postgres://postgres:password@postgres:5432/myapp
      REDIS_URL: redis://redis:6379
      NODE_ENV: development

  postgres:
    image: postgres:15
    container_name: postgres
    environment:
      POSTGRES_PASSWORD: password
      POSTGRES_USER: postgres
      POSTGRES_DB: myapp
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7
    container_name: redis
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Key Points

1. **Use `build`** for custom applications during development
2. **Use `image`** for pre-built services (databases, caches)
3. **Don't expose** all ports - only what needs external access
4. **Use service names** for internal communication
5. **Rebuild with `--build`** when code changes
6. **Internal services** don't need port mappings

## Key-takeaways

Using the `build` configuration in Docker Compose:

- Simplifies development workflow
- Automatically builds from your Dockerfile
- Integrates with other services seamlessly
- Handles network communication automatically
- Makes multi-container development efficient

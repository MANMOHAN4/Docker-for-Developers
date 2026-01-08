# Volumes in Docker Compose

## The Data Persistence Problem

When using Docker Compose, containers are temporary by default. When you run `docker compose down`, all containers are removed and any data stored inside them is lost.

This is a critical problem for stateful services like databases.

## Solution: Docker Compose Volumes

Docker Compose allows you to define persistent volumes that survive container restarts and removal.

### Creating Volumes in Compose

```yaml
version: "3.8"

services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

### How It Works

1. **Volume Definition**: `postgres_data` is a named volume managed by Docker
2. **Mount Point**: `/var/lib/postgresql/data` is where PostgreSQL stores data inside the container
3. **Persistence**: When the container stops, the volume persists
4. **Reuse**: When you restart the container, the same volume is mounted

### Practical Example

```yaml
version: "3.8"

services:
  postgres:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: myapp

  redis:
    image: redis:7
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

### Volume Lifecycle

1. **docker compose up**: Volume is created (if it doesn't exist) and mounted
2. **Container running**: Data is written to the volume
3. **docker compose down**: Container is removed, but volume persists
4. **docker compose up**: Same volume is mounted to a new container
5. **Data intact**: All data from previous container is available

## Verifying Volumes

```bash
docker volume ls
```

Shows all volumes on your system.

```bash
docker volume inspect volume_name
```

Shows details about a specific volume.

## Multiple Services, Same Volume

You can share a volume between multiple services:

```yaml
services:
  api:
    image: my_api
    volumes:
      - shared_data:/app/data

  worker:
    image: my_worker
    volumes:
      - shared_data:/worker/data

volumes:
  shared_data:
```

Both `api` and `worker` share access to the same `shared_data` volume.

## Volume Cleanup

Volumes are NOT automatically deleted when you run `docker compose down`.

### Remove Volumes

```bash
docker compose down -v
```

The `-v` flag also removes volumes.

Or manually:

```bash
docker volume rm volume_name
docker volume prune  # Remove unused volumes
```

## Best Practices

1. **Always use volumes** for stateful services (databases, caches)
2. **Name volumes meaningfully**: `postgres_data`, `redis_cache`, etc.
3. **Document volume mounts** in comments
4. **Use the `-v` flag** when removing containers if you want to clean up volumes
5. **Be careful with cleanup** - don't accidentally delete production data

## Combining Volumes with Environment Variables

```yaml
services:
  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: secure_password
      POSTGRES_USER: appuser
      POSTGRES_DB: production

volumes:
  postgres_data:
```

## Real-World Example

```yaml
version: "3.8"

services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
      - redis

  db:
    image: postgres:15
    volumes:
      - postgres_data:/var/lib/postgresql/data
    environment:
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: myapp

  redis:
    image: redis:7
    volumes:
      - redis_data:/data

volumes:
  postgres_data:
  redis_data:
```

## Key-takeaways

Docker Compose volumes:

- Persist data beyond container lifecycle
- Are managed by Docker
- Can be shared between services
- Require explicit cleanup if needed
- Are essential for any stateful service
- Provide a simple way to manage persistent storage

Without volumes, every `docker compose down` would mean losing all your data. Volumes are fundamental to using Docker Compose for anything beyond development testing.

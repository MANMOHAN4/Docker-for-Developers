# Networking in Docker Compose

## Automatic Network Creation

By default, Docker Compose automatically sets up a single network for your application. This is one of the most powerful features of Docker Compose.

### Default Network Behavior

When you run `docker compose up`, Docker Compose:

1. Creates a network automatically (named `[project_name]_default`)
2. Connects all containers to this network
3. Enables service-to-service communication by default

### How It Works

Each service defined in your `docker-compose.yml` file:

- Gets a container in the network
- Can communicate with other containers on the same network
- Can reference other services by their service name

### Example

```yaml
version: "3.8"

services:
  web:
    image: nginx

  db:
    image: postgres:15
```

When you run `docker compose up`:

- A network named (for example) `myapp_default` is created
- A web container joins this network
- A db container joins the same network
- The web service can reference the database as `db:5432`
- The db service can reference web as `web`

### Verifying Networks

```bash
docker network ls
```

You'll see the auto-created network listed as a bridge type.

## Service-to-Service Communication

### Using Service Names

Containers on the same user-defined bridge network (which is what Docker Compose creates) can communicate using service names:

```
postgres://db:5432
redis://redis:6379
api://api:8000
```

No need for IP addresses - service names are automatically resolved through Docker's embedded DNS server.

### Example Application

```yaml
services:
  api:
    image: my_api
    environment:
      DATABASE_URL: postgres://db:5432/myapp
      REDIS_URL: redis://redis:6379

  db:
    image: postgres:15

  redis:
    image: redis:7
```

The `api` service can connect to `db` and `redis` by their service names.

## Creating Custom Networks

While the default network works well, you can also create custom networks for more control.

### Defining Networks

```yaml
version: "3.8"

services:
  web:
    image: nginx
    networks:
      - frontend

  api:
    image: my_api
    networks:
      - frontend
      - backend

  db:
    image: postgres:15
    networks:
      - backend

networks:
  frontend:
    driver: bridge
  backend:
    driver: bridge
```

### Network Isolation

In this example:

- `web` can only communicate with `api`
- `api` can communicate with both `web` and `db`
- `db` can only communicate with `api`
- `web` and `db` cannot communicate directly

This provides better security and isolation between service groups.

## Common Network Drivers

- **bridge**: Default driver (recommended for Compose)
- **host**: Uses the host machine's network (not for Compose)
- **none**: Disables all networking
- **overlay**: For Docker Swarm (advanced)

## Best Practices

1. **Default Network**: For simple applications, use the default auto-created network
2. **Custom Networks**: For complex applications with multiple service groups, define custom networks
3. **Service Names**: Always use service names for inter-service communication
4. **No Port Mapping**: Services on the same network don't need port mappings to communicate
5. **External Access**: Only expose ports for services that need external access

## Key-takeaways

Docker Compose networking automatically:

- Creates networks for you
- Enables service-to-service communication by name
- Provides DNS resolution for service names
- Handles network creation and cleanup
- Allows custom network definitions when needed

This makes it easy to create multi-container applications without manually managing network connections.

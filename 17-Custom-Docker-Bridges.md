# Creating and Using Custom Docker Bridges for Networking

## Beyond the Default Bridge

While the default bridge network works, it has limitations. The best practice is to create **user-defined bridge networks** for your containerized applications.

## Creating a Custom Bridge Network

```bash
docker network create my_network
```

This creates a new bridge network named `my_network`.

### Verifying Network Creation

```bash
docker network ls
```

You'll see your custom network listed with type `bridge`.

## Running Containers on Custom Networks

To run a container on your custom network:

```bash
docker run -it \
  --network my_network \
  --name container1 \
  busybox
```

The `--network` flag specifies which network to connect to.

## Multiple Containers on Same Network

```bash
docker run -it -d --network my_network --name container1 busybox
docker run -it -d --network my_network --name container2 busybox
docker run -it -d --network my_network --name container3 nginx
```

Now you have three containers connected to the same custom network.

## DNS Resolution

The key advantage of user-defined bridges: **Containers can communicate by name**.

### Example

From inside container1:

```bash
ping container2           # Works! Uses DNS
wget http://container3    # Can access container3 by name
```

### How It Works

Docker's embedded DNS server:

- Runs on the default gateway (127.0.0.11:53)
- Maintains a mapping of container names to IP addresses
- Resolves container names to IP addresses automatically

## Inspecting Custom Networks

```bash
docker network inspect my_network
```

Shows:

- Network ID and name
- Connected containers
- IP addresses assigned to each container
- Gateway address
- Subnet information

### Example Output

```json
{
  "Name": "my_network",
  "Driver": "bridge",
  "Containers": {
    "container1_id": {
      "Name": "container1",
      "IPv4Address": "172.18.0.2/16"
    },
    "container2_id": {
      "Name": "container2",
      "IPv4Address": "172.18.0.3/16"
    }
  }
}
```

## Advantages of User-Defined Bridges

1. **DNS by Name**: Containers can ping each other by name, not just IP
2. **Better Isolation**: Only containers explicitly connected can communicate
3. **Flexible Connection**: Add/remove containers from the network without restart
4. **Network Configuration**: Control over subnet, gateway, and other settings
5. **Security**: Better control over which containers can talk to each other

## Practical Example: Web Application Stack

```bash
# Create network
docker network create app_network

# Run database
docker run -it -d \
  --network app_network \
  --name postgres \
  postgres:15

# Run Redis
docker run -it -d \
  --network app_network \
  --name redis \
  redis:7

# Run your application
docker run -it -d \
  --network app_network \
  --name app \
  my_app_image
```

Your application can now:

```
postgres://postgres:5432    # Connect to database by name
redis://redis:6379          # Connect to Redis by name
```

## Removing Networks

To remove a network:

```bash
docker network rm my_network
```

**Note**: All containers must be stopped and disconnected before removing the network.

## Best Practices

1. **Always create custom networks** for production applications
2. **One network per application** for isolation
3. **Use descriptive names** for networks (e.g., `ecommerce_network`)
4. **Document network topology** in your architecture
5. **Use DNS names** instead of IP addresses in configurations
6. **Never use the default bridge** for multi-container apps

## Key-takeaways

User-defined bridge networks are the standard approach for Docker networking:

- Create with `docker network create`
- Containers communicate by name (DNS)
- Better isolation and control
- Essential for multi-container applications
- Use with Docker Compose for production applications

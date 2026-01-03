# Understanding Docker Bridge Networking for Containers

## Networking in Docker

Docker containers have networking enabled by default, allowing them to communicate with each other and the outside world. This happens automatically without special configuration.

## Testing Container Connectivity

When you run a container, it can access the internet:

```bash
docker run -it busybox
# Inside the container:
ping google.com       # Works!
```

The container can reach external websites even though it's an isolated environment. How is this possible?

## Docker Network Drivers

Docker provides several networking modes (drivers):

- **bridge**: Default network driver
- **host**: Direct access to host networking
- **none**: No networking
- **overlay**: Advanced use case for Docker Swarm
- **ipvlan**: Advanced IP address control

## Viewing Available Networks

```bash
docker network ls
```

This shows all networks on your system. By default, you'll see:

- **bridge**: Default bridge network
- **host**: Host network mode
- **none**: No network mode

## How Bridge Networking Works

### Architecture

**Host Machine**: Your local computer with WiFi/Ethernet connection to router

**Docker Engine**: Installed on your host machine

**Docker Bridge Network**: Virtual network interface created by Docker

**Containers**: Connected to the bridge network

### The Bridge Network

When Docker is installed, it creates a default **bridge network**. This bridge:

- Acts as a virtual network switch
- Has its own IP address (usually 172.17.0.0/16)
- Provides a gateway for containers to access the internet

### Container Network Configuration

Each container gets:

- **Its own IP address** (e.g., 172.17.0.2)
- **Virtual network interface** connected to the bridge
- **Access to the gateway** for internet connectivity
- **DNS resolution** for name-to-IP mapping

## Inspecting the Bridge Network

```bash
docker network inspect bridge
```

This shows:

- Network ID
- Created date
- IP configuration
- Connected containers and their IP addresses

### Example Output

```
"Containers": {
  "abc123...": {
    "Name": "my_container",
    "IPAddress": "172.17.0.2"
  }
}
```

## Running Containers on the Default Bridge

```bash
docker run -it -d --name my_container busybox
docker network inspect bridge
```

You'll see your container listed with its assigned IP address.

## Multiple Containers Networking

Containers connected to the same bridge can communicate with each other using:

- **IP addresses**: 172.17.0.2, 172.17.0.3, etc.
- **Container names** (with limitations on default bridge)

### Important Limitation

On the **default bridge network**, containers cannot communicate with each other by container name. You must use IP addresses.

However, on **user-defined bridge networks** (covered in a later section), DNS resolution by container name works.

## Outbound Connectivity

All containers connected to the bridge can:

- Ping external websites
- Make HTTP requests
- Access any external service
- Use DNS for domain resolution

This works because the bridge acts as a gateway, forwarding traffic from the container to the host's network interface.

## Container Isolation

Despite being able to access the internet, containers are still isolated:

- Each container has its own filesystem
- Each container has its own processes
- Each container has its own IP address
- Containers cannot access each other's files or processes

The only thing shared is network connectivity through the bridge.

## Key Concepts

- **Docker creates a bridge network** automatically when installed
- **Each container gets an IP address** on this network
- **The bridge acts as a gateway** to the outside world
- **Containers can communicate externally** but are isolated from each other
- **Default bridge has limitations** for container-to-container communication (no DNS by name)

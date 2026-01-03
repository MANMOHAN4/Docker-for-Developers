# Understanding and Implementing Port Mapping in Docker

## The Port Mapping Mystery

After learning about Docker images, containers, custom images, and Dockerfiles, there is still one unsolved puzzle: why the server is not accessible from your local machine when running inside a Docker container.

### The Problem

You have a Docker container with a Node application running on port 8000. However, when you try to access it on your local machine, it doesn't work. Why?

The issue is similar to this scenario: If your friend is running an application on port 8000 on his laptop, can you access it from your laptop? No, because that application is running locally on his machine, not your machine.

Similarly, when a container runs an application on port 8000 inside the container, you cannot access it on your local machine because the container is isolated from your host machine.

### Host Machine and Containers

Your local machine is referred to as the **host machine**. The MacOS, Windows, or Linux operating system running on your computer is the host. Containers running on this host machine are isolated environments.

Since containers are isolated, you cannot directly access ports running inside them from the host machine.

## Port Mapping Solution

To access a port inside a container from your host machine, you need to perform **port mapping**.

### Port Mapping Concept

Port mapping creates a bridge between a port on the host machine and a port inside the container.

**Syntax**:

```bash
docker run -p [host_port]:[container_port] [image]
```

**Example**:

```bash
docker run -it -p 8000:8000 my_app
```

This command says: "Take the port 8000 inside the container and map it to port 8000 on the host machine."

### How It Works

When you specify `-p 8000:8000`:

- The first `8000` is the host machine port
- The second `8000` is the container port
- Any traffic to `localhost:8000` on your host is forwarded to port 8000 inside the container

### Practical Example

You can map different ports:

```bash
docker run -it -p 3000:8000 my_app
```

This maps:

- Port 3000 on your host machine
- To port 8000 inside the container

Now you would access the application at `localhost:3000` on your host machine, which connects to port 8000 inside the container.

## Important Port Mapping Rules

The port you map to on the **host machine** (first number) can be any available port. However, the port you map to inside the **container** (second number) must match the port where your application is running inside the container.

### Example of Incorrect Mapping

If your application runs on port 8000 inside the container, but you map it like this:

```bash
docker run -it -p 3000:9000 my_app
```

This will NOT work because:

- You're telling Docker to expose port 9000 from the container
- But your application runs on port 8000 inside the container
- Port 9000 has nothing running on it

The container port (second number) must match where your application actually runs.

## Multiple Port Mapping

If your application exposes multiple ports, you can map multiple ports:

```bash
docker run -it -p 3000:8000 -p 3001:9000 -p 3002:8001 my_app
```

This maps:

- Host port 3000 → Container port 8000
- Host port 3001 → Container port 9000
- Host port 3002 → Container port 8001

## Verifying Port Mapping

You can view port mappings visually through Docker Desktop:

- Open Docker Desktop
- Go to Containers
- Click on your running container
- You'll see the port mapping information displayed

You can also verify via command line:

```bash
docker ps
```

The ports column shows the mappings like `3000->8000/tcp`.

## Detached Mode with Port Mapping

When running a server with port mapping, you often want to run it in detached mode so your terminal is not blocked:

```bash
docker run -it -d -p 3000:8000 my_app
```

The `-d` flag runs the container in detached mode, allowing you to continue using your terminal while the container runs in the background.

## Key-takeaways

Port mapping is essential for accessing containerized applications:

1. **Port mapping syntax**: `-p host_port:container_port`
2. **Container port** must match where the application actually runs inside
3. **Host port** can be any available port on your machine
4. **Multiple mappings** can be specified with multiple `-p` flags
5. **Detached mode** (`-d`) keeps your terminal free when running servers

Without port mapping, containerized applications would be completely inaccessible from your host machine due to container isolation.

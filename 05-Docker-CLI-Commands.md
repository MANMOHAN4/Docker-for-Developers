# Introduction to Docker CLI and Commonly Used Commands

## Docker Engine and Command Line Interface

To run Docker, you need Docker Engine. The Docker Engine is responsible for handling the creation and management of containers and images. It:

- Creates containers
- Deletes containers
- Creates images
- Pulls images from registries

The Docker Engine is a core component that manages all these container and image operations.

## Accessing Docker Engine Through CLI

One of the great features of Docker Engine is that it exposes APIs that you can communicate with. The command-line interface (CLI) is one way to interact with the Docker Engine.

The Docker CLI is the primary method to interact with Docker Engine, especially when you're working on servers where a graphical interface is not available. When you're connected to an actual production server via SSH, you won't have Docker Desktop UI available. Therefore, you must learn the command-line commands to interact with Docker Engine.

## Docker Desktop UI Connection

Interestingly, the Docker Desktop UI that provides a beautiful graphical interface is also interacting with the same Docker Engine through APIs. Docker Desktop is just a user-friendly wrapper around the same Docker Engine that the CLI uses.

## Common Docker Commands

### Docker Help Command

To see all available commands, you can use:

```bash
docker
```

This displays all the commands you can use with Docker.

### Docker PS - Listing Containers

**Command**: `docker ps`

The `docker ps` command lists all running containers.

**Output**:
- Container ID
- Image name
- Status
- Ports
- Container name
- And other details

**Without `docker ps` (no containers running)**:
If you run `docker ps` and no containers are running, you get empty output (no running containers to display).

**Listing All Containers (including stopped)**:

```bash
docker ps -a
```

or

```bash
docker ps --all
```

By default, `docker ps` only shows running containers. The `-a` or `--all` flag shows all containers, including those in an exited state.

### Docker Images - Listing Images

**Command**: `docker images`

The `docker images` command lists all the images you have on your machine.

**Output**:
- Repository name
- Tag
- Image ID
- Size
- Creation date

**Aliases**:
You can also use:
```bash
docker image ls
```
or
```bash
docker image list
```

These are equivalent commands.

## Running Containers with Docker Run

**Command**: `docker run`

The `docker run` command creates and runs a new container from an image.

**Example**:
```bash
docker run -it ubuntu
```

When you run this command:
- Docker checks if the `ubuntu` image exists locally
- If not, it downloads the image from hub.docker.com
- It creates a new container from this image
- It connects you to the container with an interactive terminal

The `-it` flags mean:
- `-i` (interactive): Keeps stdin open even if not attached
- `-t` (tty): Allocates a pseudo-terminal

## Getting Help for Commands

To get help for any Docker command, use the `--help` flag:

```bash
docker ps --help
```

or

```bash
docker ps -h
```

This shows you all the available options and flags for that specific command.

### Exploring Help Output

The help output shows:
- Description of what the command does
- All available flags and options
- Aliases for the command

## Docker Version Command

**Command**: `docker version`

This command shows:
- Your Docker client version
- Your Docker server/engine version

It's useful for verifying that Docker is installed and running correctly.

## Docker Info Command

**Command**: `docker info`

This command displays system-wide information about Docker:
- Number of containers (running and stopped)
- Total images
- Storage information
- System configuration
- And other details

## Command Structure

Most Docker commands follow this pattern:

```bash
docker [management command] [command] [options] [arguments]
```

Examples:
```bash
docker container ls
docker image ls
docker ps
```

## Key Points About Docker CLI

### Learning Commands

Don't worry about memorizing all commands. Key points:

1. You don't need to memorize every command
2. Use `--help` to find available options
3. Most commands follow logical patterns
4. With regular use, you'll naturally remember the commonly used ones
5. It's not rocket science—just learning the tools to interact with Docker

### CLI vs UI

While Docker Desktop UI is convenient for development:
- You won't always have access to it
- Production servers typically only have CLI
- You must learn and be comfortable with CLI commands
- CLI is more powerful and flexible for automation

### Docker Aliases

Docker provides short aliases for common commands:
- `docker ps` is an alias for `docker container ls`
- `docker images` is an alias for `docker image ls`

These aliases exist because listing containers and images are very common operations.

## Summary

The Docker CLI is your primary tool for interacting with Docker Engine. Through the CLI, you can:
- List containers (`docker ps`)
- List images (`docker images`)
- Create and run containers (`docker run`)
- Get help for any command (`--help` flag)
- View version information (`docker version`)
- View system information (`docker info`)

Learning these basic commands is the foundation for working effectively with Docker. As you use Docker regularly, these commands will become second nature.
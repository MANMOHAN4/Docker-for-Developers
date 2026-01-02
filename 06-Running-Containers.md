# Running Docker Containers Using the CLI with Practical Examples

## Understanding the docker run Command

The `docker run` command is one of the most powerful and important Docker commands. It creates and runs a new container from an image.

## The docker run Syntax

```bash
docker run [flags] [image] [command]
```

## The -i Flag: Interactive Mode

The `-i` flag stands for **interactive**. It keeps stdin (standard input) open even if the container is not attached.

### How -i Works

When you run a container with `-i` flag:
- The standard input channel remains open
- You can send input to the container
- You can execute commands inside the container

### Example: Running Without -i

```bash
docker run ubuntu
```

If you run this command without the `-i` flag, you'll notice that:
- The container starts
- The connection closes immediately
- You lose the connection to the container
- You're back to your local machine's terminal

This is because without `-i`, the standard input is not kept open, so the connection breaks.

### Example: Running With -i

```bash
docker run -i ubuntu
```

With the `-i` flag:
- Standard input remains open
- You can execute commands like `ls`, `pwd`, `cat`, `touch`
- The connection stays active
- You can interact with the container

## The -t Flag: TTY Allocation

The `-t` flag stands for **TTY** (Teletype). It allocates a pseudo-terminal.

### What Does -t Do?

The `-t` flag means:
- Allocate a pseudo TTY (terminal)
- Connect your machine's terminal to the container's terminal
- Create a proper terminal experience inside the container

### The Difference Between -i and -t

**With only -i flag**:
- You can send input to the container
- But it's not a true terminal experience
- Standard input is kept open
- You can execute commands

**With -t flag**:
- You get an actual terminal
- Proper terminal prompt appears
- Better formatting and display
- True interactive terminal experience

## Combining Flags: -it

The most common usage combines both flags:

```bash
docker run -it ubuntu
```

**What -it means**:
- `-i`: Keep standard input open
- `-t`: Allocate a pseudo-terminal

**Result**: You get a full interactive terminal experience inside the container where you can:
- See a proper terminal prompt (e.g., `root@container-id#`)
- Execute commands normally
- Interact with the container like a regular terminal
- See formatted output

## Practical Examples

### Example 1: Running Interactive Ubuntu

```bash
docker run -it ubuntu
```

This gives you an interactive Ubuntu terminal where you can:
- Run `ls` to list files
- Run `pwd` to see current directory
- Create files, edit them, run programs
- Interact fully with the Ubuntu environment

### Example 2: Running a Specific Command

You can override the default command by specifying a command at the end:

```bash
docker run ubuntu ls
```

This runs the `ls` command in Ubuntu and exits.

### Example 3: Named Containers

You can give your container a specific name:

```bash
docker run -it --name my_container ubuntu
```

The `--name` flag allows you to specify a name for the container instead of getting a randomly generated one.

### Example 4: Running Commands from Other Images

You can run containers from other images:

```bash
docker run -it busybox
```

BusyBox is a lightweight image with minimal tools.

### Example 5: Running Specific Commands

You can specify exact commands to run:

```bash
docker run -it busybox ping google.com
```

This runs the `ping` command directly without keeping an interactive session.

## Understanding the Image Parameter

The last parameter in the `docker run` command is the image name:

```bash
docker run -it ubuntu
```

Here, `ubuntu` is the image name.

## Entry Points and Default Commands

Every Docker image has a default **entry point** - a command that runs when the container starts.

### Finding the Default Command

You can see the default command using:

```bash
docker image inspect ubuntu
```

Looking at the output, you'll find:
- **Cmd**: The default command to execute (e.g., `/bin/bash` for Ubuntu)
- **Entrypoint**: The entry point for the image

For Ubuntu, the default command is `bash`.

### Overriding Default Commands

You can override the default command by specifying a different one:

```bash
docker run ubuntu ls
```

This runs `ls` instead of `bash`.

You can also override with other commands:

```bash
docker run ubuntu pwd
```

This runs `pwd` (print working directory).

## Managing Running Containers

### Listing Containers

```bash
docker ps
```

Shows only running containers.

```bash
docker ps -a
```

Shows all containers (running and exited).

### Killing a Container

If a container is running a long-process (like `ping`), you can stop it:

```bash
docker kill <container-id>
```

This terminates the container process.

### Container Status

After killing, a container shows status as **killed** or **exited** when you run `docker ps -a`.

## Removing Containers

### Difference Between Kill and Remove

- **Kill**: Stops the container but the data remains
- **Remove**: Completely deletes the container and all its data

### Removing a Container

```bash
docker container rm <container-id>
```

or the shorthand:

```bash
docker rm <container-id>
```

After removal, you can verify with `docker ps -a` - the container no longer appears.

## Practical Workflow

1. **Run a container**: `docker run -it ubuntu`
2. **Execute commands**: Inside the container, run any commands
3. **Exit**: Press `Ctrl+D` or type `exit`
4. **Check status**: `docker ps -a` to see exited containers
5. **Remove if needed**: `docker rm <container-id>`

## Key Takeaways

- **docker run**: Creates and runs a new container
- **-i flag**: Keeps stdin open for interaction
- **-t flag**: Allocates a pseudo-terminal
- **-it**: Combines both for full interactive terminal
- **--name**: Gives a custom name to the container
- **Image name**: The image to run (ubuntu, busybox, etc.)
- **Command**: Optional command to execute (defaults to image's entry point)

The `docker run` command with `-it` flags is one of the most fundamental commands you'll use regularly when working with Docker containers.
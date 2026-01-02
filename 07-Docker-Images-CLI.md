# Working with Docker Images Through Command Line Interface

## Docker Image Commands

To work with Docker images through the command line, you can use the `docker image` command with various subcommands.

### Getting Help with Image Commands

To see all available image commands:

```bash
docker image --help
```

This displays all the commands and options available for managing Docker images.

## Available Image Commands

Some of the key Docker image commands include:

- `build`: Build an image from a Dockerfile
- `history`: Show the history of an image
- `import`: Import contents from a tarball
- `inspect`: Display detailed information about an image
- `load`: Load an image from a tar archive
- `ls`: List images
- `prune`: Remove unused images
- `pull`: Download an image from a registry
- `push`: Upload an image to a registry
- `rm`: Remove one or more images
- `save`: Save one or more images to a tar archive
- `tag`: Create a tag for an image

## Detailed Image Operations

### Docker Image Inspect

**Command**: `docker image inspect [image_name]`

The `inspect` command displays detailed metadata about an image:

**Example**:
```bash
docker image inspect alpine
```

**Output includes**:
- Image digest
- Creation date and time
- Configuration details
- Environment variables
- Default command
- Architecture information
- Graph driver information
- And much more metadata

This command gives you comprehensive information about how an image is configured.

### Docker Image LS (List Images)

**Command**: `docker image ls`

or simply:

```bash
docker images
```

Both commands list all images on your machine.

**Output columns**:
- REPOSITORY: Image name
- TAG: Version tag
- IMAGE ID: Unique identifier
- CREATED: When the image was created
- SIZE: Image size

**Adding flags**:
```bash
docker images -a
```

This shows all images, including intermediate images.

### Docker Image Pull

**Command**: `docker image pull [image_name]`

or:

```bash
docker pull alpine
```

This downloads an image from a registry (like Docker Hub) to your local machine.

### Docker Image RM (Remove)

**Command**: `docker image rm [image_name]`

Removes an image from your local machine.

**Important Note**: You cannot remove an image if a container is using that image. You must first remove or stop the container using that image.

**Example workflow**:

1. List containers:
```bash
docker ps -a
```

2. If a container is using the image, remove it first:
```bash
docker container rm [container_id]
```

3. Now remove the image:
```bash
docker image rm [image_name]
```

### Docker Image Prune

**Command**: `docker image prune`

This command removes all dangling images (images that are not tagged and are not used by any container).

**Important**: You can check if there are any dangling images before running prune. If there are no dangling images, the command will report that.

## Image Relationships with Containers

### Understanding Image Dependencies

An important concept to understand:
- A Docker image can have multiple containers running from it
- If a container is using an image, you cannot remove that image
- You must remove or stop the dependent container first before removing the image

### Workflow for Removing Images

1. **Check if image is in use**:
```bash
docker ps -a
```

2. **If containers exist, remove them**:
```bash
docker container rm [container_id]
```

3. **Now remove the image**:
```bash
docker image rm [image_name]
```

## Docker Image Aliases

Docker provides convenient shorthand commands:

- `docker images` is an alias for `docker image ls`
- `docker image ls` and `docker image list` are equivalent

Both show the same output:
```bash
docker images
docker image ls
docker image list
```

## Using Docker Image Tag

**Command**: `docker image tag [source_image] [target_image]`

The `tag` command allows you to create an alias for an image. This is useful when:
- You want to give an image another name
- You want to version an image
- You want to prepare an image for pushing to a registry

Later, when pushing images to cloud registries or Docker Hub, you'll use tagging to properly identify your images.

## Using Docker Image Save

**Command**: `docker image save [image_name] -o [filename.tar]`

The `save` command exports an image to a TAR archive file.

This creates a compressed/archived version of your image that can be:
- Backed up
- Transferred to another machine
- Shared with others

## Practical Workflow Example

### Step 1: List all images
```bash
docker images
```

### Step 2: Inspect an image
```bash
docker image inspect alpine
```

Review the image configuration and metadata.

### Step 3: Pull a new image
```bash
docker image pull ubuntu
```

### Step 4: List images again
```bash
docker images
```

Now you should see the newly pulled Ubuntu image.

### Step 5: Remove an image (if needed)
```bash
docker image rm alpine
```

### Step 6: Clean up dangling images
```bash
docker image prune
```

## Key Commands to Remember

| Command | Purpose |
|---------|---------|
| `docker images` | List all images |
| `docker image inspect` | View detailed image information |
| `docker image pull` | Download an image |
| `docker image rm` | Remove an image |
| `docker image prune` | Remove unused images |
| `docker image tag` | Create an image alias |
| `docker image save` | Export image to tar file |
| `docker image ls` | List images (same as docker images) |

## Summary

Working with Docker images through the CLI involves:

1. **Listing** images with `docker images`
2. **Inspecting** images for metadata with `docker image inspect`
3. **Pulling** images from registries with `docker image pull`
4. **Removing** images with `docker image rm` (after removing dependent containers)
5. **Pruning** dangling images with `docker image prune`
6. **Tagging** images for organization and sharing with `docker image tag`
7. **Saving** images for backup or transfer with `docker image save`

These commands give you full control over managing Docker images on your system.
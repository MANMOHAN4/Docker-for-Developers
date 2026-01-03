# Creating and Managing Custom Named Volumes in Docker

## Docker-Managed Volumes

While host volume mounting works well, Docker also provides **named volumes** - volumes managed by Docker itself.

## Creating Custom Volumes

### Using Docker Volume Commands

```bash
docker volume create custom_data
```

This creates a volume named `custom_data` managed by Docker.

### Listing Volumes

```bash
docker volume ls
```

Shows all volumes on your system.

### Searching for Specific Volumes

```bash
docker volume ls | grep custom
```

Filters volumes by name.

## Using Volumes in Containers

### Mounting a Named Volume

```bash
docker run -it \
  -v custom_data:/home/server \
  --rm \
  ubuntu
```

The volume `custom_data` is mounted at `/home/server` inside the container.

### Creating Files in Volumes

```bash
# Inside container
cd /home/server
echo "Ubuntu file" > ubuntu.txt
# Exit container (--rm removes the container but keeps the volume)
```

The `ubuntu.txt` file persists in the `custom_data` volume.

## Accessing Volumes from Multiple Containers

One of the key advantages: multiple containers can access the same volume.

### Container 1

```bash
docker run -it \
  -v custom_data:/home/server \
  ubuntu

# Inside: create a file
echo "From Ubuntu container" > file1.txt
```

### Container 2

```bash
docker run -it \
  -v custom_data:/home/server \
  busybox

# Inside: access the same file
cat /home/server/file1.txt     # Works!
echo "From busybox" > file2.txt
```

Both containers share access to the same volume. Files created by one container are accessible to others.

### Verifying Shared Access

In Docker Desktop, go to Volumes and view the volume details:

- See all files created by different containers
- View "In use by" containers
- See file timestamps and sizes

## Advantages of Named Volumes

1. **Docker-managed**: Docker handles storage and lifecycle
2. **Shared access**: Multiple containers can use the same volume
3. **Persistence**: Data survives container deletion
4. **No host dependency**: Don't need host folder paths
5. **Clean**: Volumes are managed in one place
6. **Cleaner commands**: Use name instead of full file paths

## Inspecting Volumes

```bash
docker volume inspect custom_data
```

Shows:

- Driver (usually `local`)
- Mount point (where it's stored on host)
- Labels
- Containers currently using it

## Cleaning Up Volumes

### Removing Individual Volumes

```bash
docker volume rm custom_data
```

This deletes the volume and all its data.

### Removing Unused Volumes

```bash
docker volume prune
```

Removes all volumes not currently in use by any container.

## Best Practices

### Named Volumes vs Host Volumes

Use **named volumes** when:

- Multiple containers need to share data
- You don't care about specific host file paths
- Docker should manage storage
- Production deployments

Use **host volumes** when:

- Developing locally and need quick access
- Must access files on host machine
- Specific file paths matter

## Real-World Scenario: Database

```bash
# Create volume for database
docker volume create postgres_data

# Run database with persistent storage
docker run -d \
  -v postgres_data:/var/lib/postgresql/data \
  --name db \
  postgres:15

# If container crashes or is removed:
docker run -d \
  -v postgres_data:/var/lib/postgresql/data \
  --name db_new \
  postgres:15

# Data persists! New container has access to old data
```

## Key-takeaways

Named volumes are Docker-managed storage:

1. **Create**: `docker volume create volume_name`
2. **Use**: `-v volume_name:/container/path`
3. **Share**: Multiple containers access the same volume
4. **Persist**: Data survives container deletion
5. **Clean**: `docker volume prune` removes unused volumes
6. **Inspect**: `docker volume inspect volume_name` for details

Named volumes are the recommended approach for data persistence in Docker applications.

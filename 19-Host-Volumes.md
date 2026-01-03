# Attaching Host Machine Volumes to Docker Containers

## The Volume Problem

When you create files inside a Docker container, those files exist only in that container. As soon as the container is removed, the data is lost.

### Example

```bash
docker run -it --rm ubuntu
# Inside container:
cd /home/ubuntu
echo "I am a secret file" > secret.txt
cat secret.txt  # Works!
# Exit container
```

Now the container is gone, and so is your `secret.txt` file.

## The Need for Persistent Storage

You need to:

1. **Store data outside containers** so it persists
2. **Share data between containers** and your host machine
3. **Access host files** from inside containers

## Volume Mounting Concept

Volume mounting connects a folder on your host machine to a folder inside the container.

### How It Works

You can tell Docker: "Take this folder from my host machine and make it accessible inside this container at this path."

Anything written to that path inside the container actually gets written to the host machine folder.

## Mounting Host Volumes

### Syntax

```bash
docker run -it -v [host_path]:[container_path] [image]
```

### Example

```bash
docker run -it -v /Users/manmohan/downloads/my_data:/home/ubuntu/manmohan ubuntu
```

This:

- Takes `/Users/manmohan/downloads/my_data` from the host
- Mounts it at `/home/ubuntu/manmohan` inside the container
- Changes in the container are reflected on the host

### Accessing the Mounted Volume

```bash
# Inside container
cd /home/ubuntu/manmohan
ls                          # See files from host
cat secret.txt              # Read host files
echo "New file" > new.txt   # Create new file on host
```

The file `new.txt` actually gets created in `/Users/manmohan/downloads/my_data/new.txt` on your host!

## Practical Example

### Setup: Create a folder on your host

```bash
mkdir -p ~/downloads/my_data
echo "Original file" > ~/downloads/my_data/secret.txt
```

### Run container with mounted volume

```bash
docker run -it \
  -v ~/downloads/my_data:/home/ubuntu/manmohan \
  ubuntu
```

### Inside the container

```bash
cd /home/ubuntu/manmohan
ls              # See secret.txt from host
cat secret.txt  # Read the file
rm secret.txt   # Delete from container
```

### Check your host machine

The file is actually deleted from your host! This is the power (and danger) of volume mounting.

## Advantages of Volume Mounting

1. **Persistent storage**: Data survives container deletion
2. **Shared data**: Multiple containers can access the same folder
3. **Edit on host**: Modify files on your host and see changes in container
4. **Development workflow**: Edit code on host, container uses updated code
5. **Backup**: Data is stored on host, easy to backup

## Multi-line Command for Readability

Long commands are hard to read. Use backslashes to split across lines:

```bash
docker run -it \
  -v ~/downloads/my_data:/home/ubuntu/manmohan \
  --rm \
  ubuntu
```

Each line is a separate part of the command. The backslash tells bash to continue on the next line.

## Important Use Cases

### Development: Edit code locally, run in container

```bash
docker run -it \
  -v ~/myproject:/app \
  -p 8000:8000 \
  my_app_image
```

- Code on host: `~/myproject`
- Code in container: `/app`
- Edit files on host, changes immediately available in container

### Database: Persistent data storage

```bash
docker run -d \
  -v ~/db_data:/var/lib/postgresql/data \
  postgres:15
```

Database files are stored in `~/db_data` on your host, persisting across container restarts.

### Shared logs

```bash
docker run -it \
  -v ~/logs:/app/logs \
  my_app_image
```

Application logs written to `/app/logs` inside container are stored in `~/logs` on host.

## Security Considerations

- **Full access**: Container has full read/write access to mounted folder
- **Data ownership**: Files created in container might be owned by root
- **Accidental deletion**: Can delete host files from inside container
- **Permission issues**: Linux file permissions apply

## Volume vs Bind Mount

What we've discussed is technically called a **bind mount** (mounting a host folder).

Docker also has **volumes** (managed by Docker, covered separately).

**For host mounting**, use the `-v` or `--volume` flag as shown above.

## Key-takeaways

Host volume mounting enables:

1. **Persistent data** storage outside containers
2. **Shared access** to files between host and containers
3. **Development workflows** where code lives on host
4. **Data backup** by storing on the host machine

Syntax: `-v /host/path:/container/path`

Essential for any stateful application or development workflow.

# Security Best Practices for Running Docker Containers

## The Root User Problem

By default, when you run a Docker container, the application runs as the **root user** (admin). This is a security vulnerability.

### Why Running as Root is Dangerous

If your containerized application has a security vulnerability that allows code execution, an attacker gains **root privileges** inside the container. This means they can:

- Delete files
- Modify system files
- Access sensitive data
- Potentially escape the container

## Creating Non-Root Users

To mitigate this risk, create a non-root user in your Dockerfile and run the application as that user.

### Creating a User Group

```dockerfile
RUN addgroup -g 1001 nodejs
```

This creates a group named `nodejs` with group ID 1001.

### Creating a User

```dockerfile
RUN adduser -u 1001 -G nodejs nodejs
```

This creates a user named `nodejs` with user ID 1001 in the `nodejs` group.

### Setting the User

```dockerfile
USER nodejs
```

This sets the user for all subsequent commands in the Dockerfile. The application will now run as `nodejs` user, not root.

### Complete Example

```dockerfile
FROM node: 20-alpine3.19 as base

# Stage 1: Build Stuff
FROM base as builder

WORKDIR /home/build

COPY package *. json .
COPY tsconfig.json .

RUN npm install

COPY src/ src/

RUN npm run build

# Stage 2: Runner
FROM base as runner

WORKDIR /home/app

COPY -- from=builder /home/build/dist dist/
COPY -- from=builder /home/build/package *. json .

RUN npm install -- omit=dev

RUN addgroup -- system -- gid 1001 nodejs
RUN adduser -- system -uid 1001 nodejs

USER nodejs

EXPOSE 8000
ENV PORT=8000

CMD [ "npm", "start" ]
```

## Benefits of Non-Root User

- **Limited Privileges**: The nodejs user has no admin permissions
- **Reduced Risk**: Even if compromised, the attacker can't delete system files
- **Better Security**: Follows the principle of least privilege
- **Compliance**: Many security standards require non-root execution

## Environment Variables

You can set environment variables in your Dockerfile and override them at runtime.

### Setting Default Environment Variables

```dockerfile
ENV PORT=8000
```

### Passing Environment Variables at Runtime

Using `-e` flag:

```bash
docker run -it -e PORT=3000 my_app
```

This sets the PORT environment variable to 3000, overriding the default value of 8000.

### Multiple Environment Variables

```bash
docker run -it -e PORT=3000 -e NODE_ENV=production my_app
```

### Using Environment Files

If you have many environment variables, store them in a `.env` file:

```
PORT=3000
NODE_ENV=production
DATABASE_URL=postgres://...
```

Pass the env file to Docker:

```bash
docker run -it --env-file .env my_app
```

The `--env-file` flag reads all variables from the specified file and loads them into the container.

## Port Exposure

In your Dockerfile, document which ports your application uses:

```dockerfile
EXPOSE 8000
```

This documents that the application expects to run on port 8000.

## Key-takeaways

Docker security best practices include:

1. **Never run as root** - Create and use a non-root user
2. **Use environment variables** - For configuration and secrets
3. **Expose documentation** - Use EXPOSE for port documentation
4. **Pass configuration at runtime** - Use -e or --env-file flags
5. **Minimize attack surface** - Use lightweight base images
6. **Regular updates** - Keep base images patched with security updates

Security should be a fundamental part of your Docker containerization strategy from the beginning.

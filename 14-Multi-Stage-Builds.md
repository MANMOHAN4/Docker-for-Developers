# Building Optimized Multi-Stage Docker Images for Production

## The Multi-Stage Build Problem

When building applications that require compilation or building (like TypeScript, C, or Java), you often have a bloated final image containing:

- Source code files (not needed in production)
- Build tools and compilers (not needed in production)
- Dev dependencies (not needed in production)

All of these are included in the final image, making it unnecessarily large.

## Example: TypeScript Application

Consider a TypeScript application:

### Build Requirements

To build TypeScript:

```bash
npm install
npm run build  # Compiles TypeScript to JavaScript
```

During build, you need:

- TypeScript compiler
- Type definitions (@types/node, @types/express)
- Development dependencies

### Runtime Requirements

After build, you only need:

- The compiled JavaScript in the `/dist` folder
- Runtime dependencies (express, etc.)
- NOT the TypeScript compiler or type definitions

## Single-Stage Dockerfile (Inefficient)

```dockerfile
FROM node:20.17-alpine

WORKDIR /home/app

COPY package*.json .
COPY tsconfig.json .

RUN npm install

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

**Problem**: The final image includes all 367+ MB of dependencies, compilers, and source code, even though only the `dist` folder and runtime dependencies are needed in production.

## Multi-Stage Dockerfile (Optimized)

Multi-stage builds use multiple `FROM` instructions to separate build and runtime stages:

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

CMD [ "npm", "start" ]
```

## Benefits of Multi-Stage Builds

### Size Reduction

- **Before**: 367+ MB (includes everything)
- **After**: Significantly smaller (typically 50-60% reduction)

### Security

- Final image doesn't contain source code
- Build tools not present in production environment
- Reduced attack surface

### Clean Production Environment

- Only what's needed to run the application
- No compilation tools or development dependencies
- Faster container startup

## Advanced Multi-Stage Example

You can have more than two stages:

```dockerfile
# Stage 1: Download dependencies
FROM node:20.17-alpine AS dependencies

WORKDIR /app
COPY package*.json .
RUN npm install

# Stage 2: Build
FROM node:20.17-alpine AS builder

WORKDIR /app
COPY --from=dependencies /app/node_modules ./node_modules
COPY package*.json .
COPY tsconfig.json .
COPY src ./src

RUN npm run build

# Stage 3: Runtime
FROM node:20.17-alpine

WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package.json .

ENTRYPOINT ["node", "dist/index.js"]
```

## Optimization: Using --omit=dev

In the final stage:

```dockerfile
RUN npm install --omit=dev
```

The `--omit=dev` flag installs only production dependencies, excluding dev dependencies like TypeScript.

## Layer Caching with Multi-Stage

Each stage has its own layer cache:

- If builder stage hasn't changed, it's cached
- Final stage rebuilds if source changes
- Build stage can be cached and reused

## Practical Comparison

### Traditional Single-Stage

- Image size: 367 MB
- Includes: Source code, TypeScript, all dependencies, build tools
- Build time: Full rebuild every time

### Multi-Stage

- Image size: ~140 MB (60% reduction)
- Includes: Only compiled code, runtime dependencies
- Build time: Faster due to layer reuse

## When to Use Multi-Stage Builds

**Always use multi-stage builds when:**

- Building compiled languages (C, Go, Rust, TypeScript)
- You have dev vs production dependencies
- You want smaller production images
- You care about security (no source code in production)
- You need faster container startup times

**Single-stage is acceptable only when:**

- Your application doesn't require a build step
- Image size isn't a concern
- The application is simple and small

## Key-takeaways

Multi-stage Docker builds provide:

1. **Smaller images** by discarding build artifacts
2. **Better security** by removing source code and build tools
3. **Cleaner production** with only necessary files
4. **Faster deployment** due to smaller download sizes
5. **Better layer caching** for development workflows

Multi-stage builds are a best practice for any production Docker image.

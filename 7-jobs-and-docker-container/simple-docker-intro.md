# Docker Refreshments

## Dockerfile

* A **Dockerfile** is used as the blueprint/instructions to create a **Docker image**.

### Build an Image from a Dockerfile

```bash
docker build -t custom-image-name -f Dockerfile.Name ./location
```

* `-t custom-image-name` → custom name for the image.
* `-f Dockerfile.Name` → specifies which Dockerfile to use.
* `./location` → specifies the **build context** / location of the files available during the build.

If using the default `Dockerfile` in the current directory:

```bash
docker build -t custom-image-name .
```

---

## Create and Run a Container

After creating an image with `docker build`, we can create a container from that image:

```bash
docker create --name custom-container-name custom-image-name
```

This only **creates** the container; it does not start it.

To start an existing container:

```bash
docker start custom-container-name
```

Alternatively, `docker run` can **create and start** the container in one command:

```bash
docker run --name custom-container-name custom-image-name
```

With detached mode:

```bash
docker run -d --name custom-container-name custom-image-name
```

### Docker Flow

```text
Dockerfile
    ↓ docker build
Docker Image
    ↓ docker run
Docker Container
```

---

# Docker Compose

With Docker Compose, we can either:

1. Use an existing image.
2. Build an image from our Dockerfile.

## Using an Existing Image

We can define an existing image in `compose.yaml`:

```yaml
services:
  app:
    image: node:24
```

When running:

```bash
docker compose up
```

Compose will use the `node:24` image. If the image is not available locally, Docker can pull it from the configured registry.

---

## Building from a Dockerfile

We can also tell Compose to build an image from a Dockerfile:

```yaml
services:
  app:
    build: .
```

Here, `.` specifies the build context.

For example:

```text
project/
├── compose.yaml
├── Dockerfile
├── package.json
└── src/
```

With:

```yaml
services:
  app:
    build: .
```

Compose will use the `Dockerfile` in the current directory to build the image.

---

## Starting Docker Compose

```bash
docker compose up
```

This will:

```text
compose.yaml
     ↓
Check/build required images if necessary
     ↓
Create containers
     ↓
Start containers
```

If we want to explicitly rebuild the images before starting:

```bash
docker compose up --build
```

### Docker Compose Summary

```text
Existing Image:

compose.yaml
     ↓
image: node:24
     ↓
Pull image if needed
     ↓
Create Container
     ↓
Start Container
```

```text
Build from Dockerfile:

compose.yaml
     ↓
build: .
     ↓
Dockerfile
     ↓
Build Image
     ↓
Create Container
     ↓
Start Container
```

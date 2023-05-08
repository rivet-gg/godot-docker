# Godot + Docker

These Dockerfiles help get up and running with Godot 4.

## Uprading Godot

The container can be built with

```bash
docker build --build-arg GODOT_VERSION=4.0.2 .
```

[See available Godot versions](https://downloads.tuxfamily.org/godotengine/)

## Recommended usage

### Isolated build environments

Godot projects can be built quickly by mounting volumes like:

```bash
mkdir -p build/linux
docker run -v "$(pwd):/app" rivetgg/godot:4.0.2 --export-release "Linux/X11" --headless ./build/linux/game.x86_64
```

See also [godot-ci](https://github.com/abarichello/godot-ci) for Dockerfiles tailored for CI environments.

### Containerizing for production

If running Godot as a dedicated server in production, we recommend using the following Dockerfile:

```dockerfile
FROM rivetgg/godot:4.0.2 AS builder
COPY . .
RUN mkdir -p build/linux \
    && godot -v --export-release "Linux/X11" --headless ./build/linux/game.x86_64

FROM ubuntu:22.04
RUN apt update -y \
    && apt install -y expect-dev \
    && rm -rf /var/lib/apt/lists/*
COPY --from=builder /app/build/linux/ /app

# Unbuffer output so the logs get flushed
CMD ["sh", "-c", "unbuffer /app/game.x86_64 --verbose --headless -- --server | cat"]
```


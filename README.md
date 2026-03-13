# dev_container_go

[![Build](https://github.com/abitofhelp/dev_container_go/actions/workflows/docker-build.yml/badge.svg)](https://github.com/abitofhelp/dev_container_go/actions/workflows/docker-build.yml)
[![Publish](https://github.com/abitofhelp/dev_container_go/actions/workflows/docker-publish.yml/badge.svg)](https://github.com/abitofhelp/dev_container_go/actions/workflows/docker-publish.yml)
[![License: BSD-3-Clause](https://img.shields.io/badge/License-BSD--3--Clause-blue.svg)](LICENSE)
[![Go](https://img.shields.io/badge/Go-1.26.1-00ADD8)](#pre-installed-tools)
[![Container](https://img.shields.io/badge/container-ghcr.io%2Fabitofhelp%2Fdev--container--go-0A66C2)](#image-name)

Professional Go development container for cloud platform and desktop development
with protobuf/gRPC and Bazel (Bzlmod) support.

## Supported Architectures

The image uses Ubuntu 24.04 and supports amd64 + arm64 multi-arch builds.

### Component Sources

| Component | Source | amd64 | arm64 | Version |
|-----------|--------|:-----:|:-----:|---------|
| Base | Ubuntu 24.04 | Y | Y | glibc 2.39 |
| Go | go.dev tarball | Y | Y | 1.26.1 |
| GCC | apt (build-essential) | Y | Y | 13 |
| protoc | apt (protobuf-compiler) | Y | Y | system |
| protoc-gen-go | go install | Y | Y | latest |
| protoc-gen-go-grpc | go install | Y | Y | latest |
| buf | GitHub release | Y | Y | 1.50.0 |
| Bazelisk | GitHub release | Y | Y | v1.25.0 |
| gopls | go install | Y | Y | latest |
| dlv (Delve) | go install | Y | Y | latest |
| staticcheck | go install | Y | Y | latest |
| golangci-lint | binary installer | Y | Y | latest |

### Cross-Compilation Targets

Go cross-compilation is built-in — no additional cross-compilers needed.

| Target | GOOS | GOARCH | Notes |
|--------|------|--------|-------|
| macOS Intel | darwin | amd64 | Pure Go only |
| macOS Apple Silicon | darwin | arm64 | Pure Go only |
| Linux x86_64 | linux | amd64 | Native + CGo |
| Linux ARM64 | linux | arm64 | Native + CGo |
| Windows x86_64 | windows | amd64 | Pure Go only |

CGo cross-compilation requires a C cross-compiler for the target platform.
Pure Go binaries cross-compile without additional tooling.

### Verified Test Matrix

| Image | Ubuntu VM (amd64) | macOS Intel (amd64) | MacBook Pro (arm64) |
|-------|:---:|:---:|:---:|
| `dev-container-go` | Pending | Pending | Pending |

## Image Name

```text
ghcr.io/abitofhelp/dev-container-go
```

## Why This Container Is Useful

This container provides a reproducible Go development environment that adapts
to the host user at runtime. Any developer can pull the pre-built image and
run it without rebuilding.

The included `.zshrc` detects when it is running inside a container and
visibly marks the prompt, which helps prevent common mistakes:

- editing files in the wrong terminal
- confusing host and container environments
- forgetting which toolchain path is active
- debugging UID, GID, or mount issues more slowly than necessary

Example prompt:

```text
parallels@container /workspace (main) [ctr:rootless]
❯
```

## Features

- Multi-architecture support (`linux/amd64` + `linux/arm64`)
- Go 1.26.1 from the official tarball (SHA256 verified)
- Built-in cross-compilation for macOS, Linux, and Windows
- Protobuf/gRPC development: protoc, protoc-gen-go, protoc-gen-go-grpc
- Protobuf linting and management: buf
- Bazel build system via Bazelisk (Bzlmod)
- Go development tools: gopls, dlv, staticcheck, golangci-lint
- GCC for CGo compilation
- Python 3 + venv
- Zsh interactive shell
- Runtime-adaptive user identity (no rebuild needed per developer)
- Container-aware shell prompt
- Designed for nerdctl + containerd (rootless)
- Also works with Docker (rootful), Podman (rootless), and Kubernetes
- GitHub Actions for build verification and container publishing
- Makefile for common build and run targets

## Pre-installed Tools

| Category | Tools |
|----------|-------|
| **Go toolchain** | go, gofmt |
| **Go tools** | gopls, dlv (Delve), staticcheck, golangci-lint |
| **Protobuf/gRPC** | protoc, protoc-gen-go, protoc-gen-go-grpc, buf |
| **Build system** | Bazelisk (as bazel), make |
| **C compiler** | gcc, g++ (for CGo) |
| **Version control** | git, patch, openssh-client (ssh, scp) |
| **Text processing** | awk, sed, grep, diff, find, xargs, sort, uniq, wc, head, tail, tr, cut, tee |
| **Network** | curl, wget, rsync |
| **Archives** | tar, zip, unzip, xz, gzip, bzip2 |
| **Editors** | vim, nano |
| **Pagers / utilities** | less, more, file, which, lsof, ps, jq |
| **Search** | ripgrep (rg), fd-find (fdfind), fzf |
| **Python** | python3, pip3, python3-venv |
| **Shell** | zsh (default), bash, zsh-autosuggestions, zsh-syntax-highlighting |
| **Container** | gosu, sudo |

## Quick Start

### Pull a pre-built image

```bash
make pull
```

### Build from source

```bash
make build
```

### Run

```bash
cd ~/projects/my_go_app
make -f /path/to/dev_container_go/Makefile run
```

> **Note**: When using `make -f`, the Makefile mounts the caller's current
> directory (not the Makefile's directory) into the container. This is
> intentional — it bind-mounts your project, not the container repository.

The current directory is mounted into the container at `/workspace`. The
entrypoint adapts the container's home directory layout to match your host
user, so bind-mounted files are readable and writable.

### Inspect configured values

```bash
make inspect
```

## Manual Build

```bash
nerdctl build -t dev-container-go .
```

## Manual Run

```bash
nerdctl run -it --rm \
  -e HOST_UID=$(id -u) \
  -e HOST_GID=$(id -g) \
  -e HOST_USER=$(whoami) \
  -v "$(pwd)":/workspace \
  -w /workspace \
  dev-container-go
```

## Use Docker or Podman Instead of nerdctl

All Makefile targets use `CONTAINER_CLI`, which defaults to `nerdctl`. Override
it to use Docker or Podman:

```bash
make build CONTAINER_CLI=docker
make run CONTAINER_CLI=docker
```

Or use the convenience aliases:

```bash
make docker-build
make docker-run

make podman-build
make podman-run
```

Podman rootless uses `--userns=keep-id` to map the host user directly into the
container without needing the `HOST_*` environment variables or entrypoint
adaptation. Podman requires `crun` and `fuse-overlayfs`. The `--userns=keep-id`
flag requires kernel support for unprivileged private mounts (see User Guide
for details and known VM limitations).

## Cross-Compilation

Go supports cross-compilation natively. Set `GOOS` and `GOARCH` to build for
any target platform:

```bash
# macOS AMD64
GOOS=darwin GOARCH=amd64 go build -o bin/myapp-darwin-amd64 ./cmd/myapp

# macOS ARM64 (Apple Silicon)
GOOS=darwin GOARCH=arm64 go build -o bin/myapp-darwin-arm64 ./cmd/myapp

# Linux AMD64
GOOS=linux GOARCH=amd64 go build -o bin/myapp-linux-amd64 ./cmd/myapp

# Linux ARM64
GOOS=linux GOARCH=arm64 go build -o bin/myapp-linux-arm64 ./cmd/myapp

# Windows AMD64
GOOS=windows GOARCH=amd64 go build -o bin/myapp-windows-amd64.exe ./cmd/myapp
```

No additional cross-compilers are needed for pure Go code. If your project
uses CGo, you will need a C cross-compiler for the target platform.

## Bazel with Bzlmod

Bazelisk is installed as `/usr/local/bin/bazel`. It automatically downloads
the correct Bazel version based on `.bazelversion` or `MODULE.bazel` in your
project. This container supports the modern Bzlmod dependency system.

## Housekeeping

Remove build artifacts (saved images, source archives):

```bash
make clean
```

Create a compressed source archive from the current HEAD:

```bash
make compress
```

## Deployment Environments

This image supports three deployment environments with a single build.

### Local Development (nerdctl rootless)

This is the primary workflow. `make run` passes the host identity and mounts
the current directory:

```bash
cd ~/projects/my_go_app
make run
```

The entrypoint sets up the home directory layout to match your host identity.
In rootless mode, the process stays as container UID 0 (which maps to the host
user via the user namespace) for bind-mount correctness. This is safe — no
privilege escalation is possible.

### CI / Docker Rootful

The image runs as the fallback non-root user (`dev:1000:1000`) by default when
no `HOST_*` environment variables are passed. GitHub Actions workflows build
and publish the image using Docker.

### Kubernetes

The image is compatible with Kubernetes out of the box. Source code is
provisioned via PersistentVolumeClaims or init containers (e.g., git-sync),
not bind mounts.

Example pod spec:

```yaml
securityContext:
  runAsUser: 1000
  runAsGroup: 1000
  fsGroup: 1000
  runAsNonRoot: true
containers:
  - name: go-dev
    image: ghcr.io/abitofhelp/dev-container-go:latest
    workingDir: /workspace
    volumeMounts:
      - name: source
        mountPath: /workspace
volumes:
  - name: source
    persistentVolumeClaim:
      claimName: go-source
```

`fsGroup: 1000` ensures the volume is writable by the container user.
Kubernetes manifests and Helm charts are not included in this repository.
Teams should create these per their cluster policies.

## Rootless Security

In rootless container runtimes (nerdctl/containerd rootless, Podman rootless),
the container runs inside a user namespace where container UID 0 maps to the
unprivileged host user. The process cannot escalate beyond the host user's
privileges. The entrypoint script detects this and avoids dropping privileges,
because doing so would map the process to a subordinate UID that cannot access
bind-mounted host files.

| Runtime          | Container UID 0 is...  | Bind mount access via...  | Security boundary      |
|------------------|------------------------|---------------------------|------------------------|
| Docker rootful   | Real root (dangerous)  | gosu drop to HOST_UID     | Container isolation    |
| nerdctl rootless | Host user (safe)       | Stay UID 0 (= host user)  | User namespace         |
| Podman rootless  | Host user (safe)       | --userns=keep-id          | User namespace         |
| Kubernetes       | Blocked by policy      | fsGroup in pod spec       | Pod security standards |

## Version Tags

```text
ghcr.io/abitofhelp/dev-container-go:latest
ghcr.io/abitofhelp/dev-container-go:go-1.26.1
```

The included publish workflow automatically creates tags in these styles.

## GitHub Actions

This repository includes:

- `docker-build.yml` to verify the Dockerfile on every push and pull request
  (multi-arch build + amd64 smoke test)
- `docker-publish.yml` to publish the image to GitHub Container Registry
  (amd64 + arm64)
- automatic tagging based on Go version
- all actions pinned by SHA digest for supply-chain security

## Repository Layout

```text
dev_container_go/
├── .dockerignore
├── .github/
│   └── workflows/
│       ├── docker-build.yml
│       └── docker-publish.yml
├── .gitignore
├── .zshrc
├── Dockerfile              ← Go 1.26.1, protobuf, Bazelisk
├── entrypoint.sh
├── examples/
│   └── hello_go/
├── LICENSE
├── Makefile
├── README.md
└── USER_GUIDE.md
```

## License

BSD-3-Clause — see `LICENSE`.

## AI Assistance and Authorship

This project was developed by Michael Gardner with AI assistance from Claude
(Anthropic) and GPT (OpenAI). AI tools were used for design review,
architecture decisions, and code generation. All code has been reviewed and
approved by the human author. The human maintainer holds responsibility for
all code in this repository.

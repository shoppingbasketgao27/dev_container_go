<!-- ====================================================================== -->
<!-- USER_GUIDE.md                                                          -->
<!-- ====================================================================== -->
<!-- Copyright (c) 2026 Michael Gardner, A Bit of Help, Inc.               -->
<!-- SPDX-License-Identifier: BSD-3-Clause                                  -->
<!-- See LICENSE file in the project root.                                  -->
<!-- ====================================================================== -->

# User Guide: dev_container_go

**Version**: 1.0.0-rc1
**Date**: 2026-03-13
**Authors**: Michael Gardner, Claude (Anthropic), GPT (OpenAI)

---

## 1. Prerequisites

### 1.1 Primary runtime: nerdctl + containerd (rootless)

This is the default development runtime. Install nerdctl and containerd
following the [nerdctl documentation](https://github.com/containerd/nerdctl).

### 1.2 Optional: Docker Engine (rootful testing)

Docker Engine is required for `make test-docker` and rootful testing.

```bash
# Ubuntu 24.04
sudo apt-get update
sudo apt-get install -y docker.io docker-buildx

# Add your user to the docker group.
sudo usermod -aG docker "$USER"

# Apply the group change — log out and back in.
# Verify after re-login.
docker --version
docker buildx version
```

> **Do not use `newgrp docker`** as a shortcut to apply the group change.
> It sets `docker` as the primary GID, which breaks Podman's `newuidmap`
> if Podman is also installed. A full logout/login picks up `docker` as a
> supplementary group and avoids this conflict.

Docker Engine coexists safely with rootless nerdctl/containerd. Docker runs
a system-level containerd at `/run/containerd/containerd.sock`, while rootless
nerdctl runs a user-space containerd at `~/.local/share/containerd/`. They use
separate storage and do not conflict.

### 1.3 Optional: Podman (rootless testing)

Podman is required for `make test-podman`.

```bash
# Ubuntu 24.04
sudo apt-get update
sudo apt-get install -y podman
```

Podman rootless requires `crun` and `fuse-overlayfs`:

```bash
sudo apt-get install -y crun
```

Configure Podman to use `crun` and `fuse-overlayfs`:

```ini
# ~/.config/containers/containers.conf
[engine]
runtime = "crun"
```

```ini
# ~/.config/containers/storage.conf
[storage]
driver = "overlay"

[storage.options.overlay]
mount_program = "/usr/local/bin/fuse-overlayfs"
```

> **Known limitation**: Podman's `--userns=keep-id` requires kernel support
> for unprivileged private mounts. This does not work in Parallels Desktop
> VMs due to kernel restrictions on mount propagation. Testing on bare-metal
> Ubuntu or non-Parallels VMs is pending. See §12 for testing status.

---

## 2. Design Goals

1. **One image, any developer** — a pre-built image from GHCR works for any
   developer without rebuilding. User identity is provided at run time, not
   baked in at build time.
2. **Bind-mounted source** — the developer's host project directory is
   mounted into the container. Edits inside the container are live on the host.
3. **Correct file permissions** — the container process runs with the host
   user's UID/GID so that bind-mounted files are readable and writable.
4. **Works in all three target environments** — local rootless nerdctl, local
   rootful Docker, and Kubernetes.
5. **Secure by default** — non-root inside the container in rootful runtimes.
   In rootless runtimes, container UID 0 is already unprivileged on the host.

---

## 3. Architecture: Runtime-Adaptive User

The image ships with a **generic fallback user** (`dev:1000:1000`) for CI and
Kubernetes. At run time, the **entrypoint script** reads host identity from
environment variables and creates or adapts the in-container user to match.

```
Host                          Container
─────                         ─────────
$(whoami)  → HOST_USER  ───→  entrypoint.sh creates user
$(id -u)   → HOST_UID   ───→  with matching UID
$(id -g)   → HOST_GID   ───→  and matching GID
$(pwd)     → -v mount   ───→  /workspace (bind mount)
```

---

## 4. File Inventory

```
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
└── USER_GUIDE.md          ← this file
```

---

## 5. Dockerfile Design

### 5.1 Go installation

Go is installed from the official tarball downloaded from `go.dev`. The
download is architecture-aware (`dpkg --print-architecture`) and verified
with SHA256 checksums. Go is extracted to `/usr/local/go` and added to PATH.

### 5.2 Go development tools

Development tools are installed system-wide to `/usr/local/bin` using
`GOBIN=/usr/local/bin go install`. This ensures tools are available to any
user regardless of their GOPATH. The root module cache (`/root/go`) is
cleaned up after installation.

Tools installed:
- **gopls** — Go language server
- **dlv** — Delve debugger
- **staticcheck** — static analysis
- **golangci-lint** — meta-linter (installed via binary installer)
- **protoc-gen-go** — protobuf Go code generator
- **protoc-gen-go-grpc** — gRPC Go code generator

### 5.3 Protobuf and gRPC

The protobuf compiler (`protoc`) is installed from Ubuntu's `protobuf-compiler`
package. The Go plugins (`protoc-gen-go`, `protoc-gen-go-grpc`) are installed
via `go install`.

**buf** is installed from the official GitHub release binary for protobuf
linting, formatting, and breaking change detection.

### 5.4 Bazelisk and Bzlmod

Bazelisk is installed as `/usr/local/bin/bazel`. When you run `bazel`, Bazelisk
reads `.bazelversion` or `MODULE.bazel` in your project and automatically
downloads and runs the correct Bazel version.

This container supports the modern Bzlmod dependency system. Legacy WORKSPACE
files are not the recommended approach.

### 5.5 Cross-compilation

Go cross-compilation is built into the toolchain. No additional cross-compilers
are needed for pure Go code. Set `GOOS` and `GOARCH` environment variables:

| Target | GOOS | GOARCH |
|--------|------|--------|
| macOS Intel | darwin | amd64 |
| macOS Apple Silicon | darwin | arm64 |
| Linux x86_64 | linux | amd64 |
| Linux ARM64 | linux | arm64 |
| Windows x86_64 | windows | amd64 |

For CGo cross-compilation, install a C cross-compiler for the target platform.

### 5.6 Shared design elements

- Base image pinned by SHA256 digest for reproducibility.
- `ENV HOME` set before `ENV PATH` to ensure correct `${HOME}` resolution.
- `SHELL ["/bin/bash", "-o", "pipefail", "-c"]` for safe pipe handling.
- Build-time user (`dev:1000:1000`) as fallback for CI and Kubernetes.
- LICENSE, README, and USER_GUIDE copied into image at
  `/usr/share/doc/dev-container-go/`.
- Entrypoint-based runtime user adaptation.

---

## 6. Entrypoint Script (entrypoint.sh)

### 6.1 Responsibilities

1. Export container-detection environment variables (`IN_CONTAINER=1`,
   `CONTAINER_RUNTIME`) so that `.zshrc` can detect the container environment
   reliably without inspecting `/proc` or sentinel files.
2. Read `HOST_USER`, `HOST_UID`, `HOST_GID` from environment.
3. If they are set and the entrypoint is running as root:
   a. Create a group with the given GID (if it does not exist).
   b. Create or adapt a user with the given username, UID, GID, home
      directory, and shell.
   c. Copy the default `.zshrc` into the new home if it does not exist.
   d. Set ownership on the home directory.
   e. Detect whether the runtime is rootless or rootful.
   f. If rootful: drop privileges via `gosu` and exec the CMD.
   g. If rootless: stay as UID 0 (which is the host user), set
      `HOME=/home/$HOST_USER`, and exec the CMD.
4. If `HOST_*` vars are not set, fall through to the default user (`dev`)
   and exec the CMD directly.

### 6.2 Rootless detection

The entrypoint detects rootless mode by checking whether UID 0 inside the
container maps to a non-root UID on the host:

```bash
is_rootless() {
    if [ -f /proc/self/uid_map ]; then
        local host_uid
        host_uid=$(awk '/^\s*0\s/ { print $2 }' /proc/self/uid_map)
        [ "$host_uid" != "0" ]
    else
        return 1
    fi
}
```

### 6.3 Privilege drop decision

```
if running as UID 0:
    if HOST_USER/HOST_UID/HOST_GID provided:
        create/adapt user
        if rootless:
            # Container UID 0 == host user. Dropping to HOST_UID would
            # map to an unmapped subordinate UID and break bind mounts.
            export HOME=/home/$HOST_USER
            exec "$@"                          # stay UID 0
        else (rootful):
            exec gosu "$HOST_USER" "$@"        # drop to real user
    else:
        # No host identity. Fall through to default user.
        exec gosu dev "$@"
else:
    # Already non-root (e.g., K8s securityContext). Just run.
    exec "$@"
fi
```

### 6.4 Error handling

- If `HOST_UID` is set but `HOST_USER` is not, default `HOST_USER` to `dev`.
- If `HOST_GID` is not set, default to the value of `HOST_UID`.
- The entrypoint must never prevent the container from starting.
- If user/group creation fails (e.g., UID conflict), the fallback is
  deterministic and depends on the runtime:
  - **Rootless**: log a warning, stay as UID 0 (which is the host user),
    set `HOME` to the fallback user's home (`/home/dev`), and exec the CMD.
  - **Rootful**: log a warning, drop to the fallback user via `gosu dev`,
    and exec the CMD.

---

## 7. Container Detection (.zshrc)

The entrypoint script exports `IN_CONTAINER=1` and `CONTAINER_RUNTIME` as
environment variables before exec'ing the shell. The `.zshrc` checks these
directly:

```bash
# Container detection — trust the entrypoint marker first
if [[ -n "$IN_CONTAINER" ]] && (( IN_CONTAINER )); then
    :
elif [[ -f /.dockerenv ]]; then
    ...existing fallback checks...
fi
```

The existing fallback checks (`/.dockerenv`, `/run/.containerenv`,
`/proc/1/cgroup`) are kept for cases where the `.zshrc` is used outside this
image.

---

## 8. Security Model Summary

| Runtime             | Container UID 0 is... | Bind mount access via... | Security boundary        |
|---------------------|-----------------------|--------------------------|--------------------------|
| Docker rootful      | Real root (dangerous) | gosu drop to HOST_UID    | Container isolation      |
| nerdctl rootless    | Host user (safe)      | Stay UID 0 (= host user) | User namespace           |
| Podman rootless     | Host user (safe)      | --userns=keep-id         | User namespace           |
| Kubernetes          | Blocked by policy     | fsGroup in pod spec      | Pod security standards   |

---

## 9. Resolved Questions

1. **Go installation method**: Official tarball from go.dev with SHA256
   verification. Ubuntu's `golang` apt package lags behind and causes
   module compatibility friction. Single Dockerfile (no system variant).
   **Decided.**

2. **Go tool installation**: System-wide to `/usr/local/bin` via
   `GOBIN=/usr/local/bin go install`. Ensures tools are available to any
   user. golangci-lint uses the official binary installer per their
   recommendation. **Decided.**

3. **Protobuf tooling**: `protoc` from apt, Go plugins via `go install`,
   buf for linting/formatting/breaking change detection. **Decided.**

4. **Build system**: Bazelisk installed as `/usr/local/bin/bazel` for
   seamless Bzlmod workflows. Legacy WORKSPACE is not recommended.
   **Decided.**

5. **Cross-compilation**: Native Go GOOS/GOARCH. No embedded tooling.
   CGo cross-compilation documented but not pre-installed. **Decided.**

6. **gosu vs su-exec**: `gosu` — more common in Docker ecosystems, available
   in Ubuntu apt. **Decided.**

7. **Container detection**: Entrypoint exports `IN_CONTAINER=1` and
   `CONTAINER_RUNTIME` as environment variables. `.zshrc` checks those first,
   with existing sentinel/cgroup checks as fallback. **Decided.**

8. **Workspace path**: `/workspace` — fixed mount point, decoupled from
   username. **Decided.**

9. **Configurable container CLI**: `CONTAINER_CLI ?= nerdctl` with
   `docker-run` / `docker-build` as convenience aliases. **Decided.**

10. **Podman support**: Added `podman-build` and `podman-run` targets.
    `podman-run` uses `--userns=keep-id` instead of `HOST_*` environment
    variables. **Decided.**

11. **sudo + passwordless sudo**: Kept intentionally for development
    convenience. In rootless runtimes, container UID 0 is already
    unprivileged on the host. **Decided.**

## 10. Remaining Open Questions

None at this time.

---

## 11. CI Workflow Design

### 11.1 docker-build.yml

Single job that builds the image for both amd64 and arm64:

- Builds with `docker buildx build --platform linux/amd64,linux/arm64`
- Loads amd64 image for smoke test (`--load` only supports single platform)
- Smoke test compiles `examples/hello_go` with `go build`, verifies
  cross-compilation for all targets, and checks toolchain versions

### 11.2 docker-publish.yml

Single job:

- Builds and pushes `dev-container-go` for amd64+arm64

Tag scheme:
- `latest`, `go-1.26.1`, `v{tag}`

All GitHub Actions are pinned by SHA digest for supply-chain security.

---

## 12. Shell Aliases (.zshrc)

The `.zshrc` provides Go development aliases:

| Alias | Command | Description |
|-------|---------|-------------|
| `gb` | `go build ./...` | Build all packages |
| `gfmt` | `gofmt -w` | Format file in-place |
| `gln` | `golangci-lint run` | Run linter |
| `gr` | `go run .` | Run current package |
| `gt` | `go test ./...` | Run all tests |
| `gtc` | `go test -coverprofile=... && go tool cover -html=...` | Test with coverage report |
| `gtv` | `go test -v ./...` | Run all tests (verbose) |
| `gvet` | `go vet ./...` | Run vet checks |

Plus standard git, navigation, file, and search aliases.

---

## 13. Upgrading Component Versions

### 13.1 Ubuntu base image

The Dockerfile pins its base image by digest for reproducibility.

```bash
nerdctl pull ubuntu:24.04
nerdctl image inspect ubuntu:24.04 \
  | python3 -c "import json,sys; d=json.load(sys.stdin); print(d[0]['RepoDigests'][0])"
# Update the FROM line in the Dockerfile with the new digest.
```

Rebuild and test the image after updating.

### 13.2 Go version

1. Check the latest Go release at `https://go.dev/dl/`.
2. Update `ARG GO_VERSION=X.Y.Z` in `Dockerfile`.
3. Update `ARG GO_SHA256_AMD64` and `ARG GO_SHA256_ARM64` with the new
   checksums from the download page.
4. Rebuild and verify: `go version`.
5. Update the `go-X.Y.Z` tag in `.github/workflows/docker-publish.yml`.
6. Update `go X.Y` in `examples/hello_go/go.mod`.

### 13.3 Bazelisk version

1. Check the latest release at
   `https://github.com/bazelbuild/bazelisk/releases`.
2. Update `ARG BAZELISK_VERSION=vX.Y.Z` in `Dockerfile`.
3. Rebuild and verify: `file /usr/local/bin/bazel`.

### 13.4 buf version

1. Check the latest release at `https://github.com/bufbuild/buf/releases`.
2. Update `ARG BUF_VERSION=X.Y.Z` in `Dockerfile`.
3. Rebuild and verify: `buf --version`.

### 13.5 Go development tools

Go tools installed via `go install ... @latest` are updated automatically
when the image is rebuilt. To pin specific versions, replace `@latest` with
`@vX.Y.Z` in the Dockerfile.

### 13.6 golangci-lint

golangci-lint is installed via the official binary installer, which fetches
the latest version by default. To pin, add a version argument:
`sh -s -- -b /usr/local/bin vX.Y.Z`.

### 13.7 protoc

The protobuf compiler version is determined by Ubuntu's `protobuf-compiler`
package. Version updates come with Ubuntu package updates.

### 13.8 Checklist

- [ ] Update version numbers / digests in all files listed above.
- [ ] Rebuild the image: `make build-no-cache`.
- [ ] Run the image and verify toolchain versions.
- [ ] Commit, tag, and push.

---

## 14. Pre-Release Testing Status

This section tracks testing gaps that should be resolved before the next
release. Remove or update entries as they are verified.

| Area                              | Status       | Notes                                                        |
|-----------------------------------|--------------|--------------------------------------------------------------|
| Rootless nerdctl (local)          | Pending      | Not yet tested.                                              |
| Docker rootful (macOS)            | Pending      | Not yet tested.                                              |
| GitHub Actions build workflow     | Pending      | Not yet tested (no push to GitHub yet).                      |
| GitHub Actions publish workflow   | Pending      | Not yet tested (no push to GitHub yet).                      |
| Podman rootless (local)           | Blocked      | `--userns=keep-id` fails in Parallels VM (kernel restriction). |
| Kubernetes deployment             | Not tested   | Image is designed to be compatible; no cluster available.    |

---

Copyright (c) 2026 Michael Gardner, A Bit of Help, Inc.
SPDX-License-Identifier: BSD-3-Clause

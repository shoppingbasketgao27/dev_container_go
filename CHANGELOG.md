# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-03-13

### Changed

- Upgraded all GitHub Actions to Node.js 24-compatible versions:
  actions/checkout v6.0.2, docker/setup-qemu-action v4.0.0,
  docker/setup-buildx-action v4.0.0, docker/login-action v4.0.0,
  docker/metadata-action v6.0.0, docker/build-push-action v7.0.0.

## [1.0.0-rc1] - 2026-03-13

### Added

- Ubuntu 24.04 base image pinned by digest for reproducibility.
- Go 1.26.1 from official tarball with SHA256 verification (arch-aware).
- Supports `linux/amd64` and `linux/arm64` multi-arch builds.
- Go development tools installed system-wide to `/usr/local/bin`:
  gopls, dlv (Delve), staticcheck, golangci-lint.
- Protobuf/gRPC development: protoc (apt), protoc-gen-go,
  protoc-gen-go-grpc (go install).
- Buf for protobuf linting, formatting, and breaking change detection.
- Bazelisk installed as `/usr/local/bin/bazel` for seamless Bzlmod workflows.
- Native Go cross-compilation for macOS (amd64, arm64), Linux (amd64, arm64),
  and Windows (amd64) — no additional cross-compilers required.
- GCC (build-essential) for CGo compilation.
- Python 3 with venv support.
- Zsh interactive shell with autosuggestions and syntax highlighting.
- Runtime-adaptive user identity via `entrypoint.sh` — no rebuild needed
  per developer.
- Rootless detection via `/proc/self/uid_map` inspection.
- Rootful privilege drop via `gosu`.
- `DISPLAY_USER` environment variable for correct prompt identity in
  rootless mode.
- Container detection markers (`IN_CONTAINER`, `CONTAINER_RUNTIME`)
  exported by entrypoint for reliable `.zshrc` detection.
- Container-aware Zsh prompt with git branch and runtime indicator.
- Go development aliases: `gb`, `gfmt`, `gln`, `gr`, `gt`, `gtc`,
  `gtv`, `gvet`.
- `container_info` shell function for quick environment diagnostics.
- Makefile with targets: `build`, `build-no-cache`, `run`, `run-root`,
  `run-shell`, `pull`, `test`, `test-docker`, `test-podman`, `inspect`,
  `save`, `show-tags`, `tag-version`, `tag-latest`, `clean`, `compress`.
- Docker convenience aliases: `docker-pull`, `docker-build`, `docker-run`.
- Podman convenience aliases: `podman-pull`, `podman-build`, `podman-run`
  with `--userns=keep-id`.
- Configurable container CLI via `CONTAINER_CLI` variable (default: nerdctl).
- GitHub Actions build workflow with multi-arch build and smoke test
  (includes cross-compilation validation for all five targets).
- GitHub Actions publish workflow for GHCR.
- `examples/hello_go/` smoke test project with `go.mod`.
- LICENSE, README, and USER_GUIDE copied into image at
  `/usr/share/doc/dev-container-go/`.
- Comprehensive USER_GUIDE.md covering architecture, security model,
  version upgrade procedures, and design decisions.

### Security

- Base image pinned by SHA256 digest.
- Go tarball verified by SHA256 checksum (per-architecture).
- All GitHub Actions pinned by SHA digest for supply-chain security.
- `latest` tag only published on semver tags or explicit opt-in.
- `run-root` bypasses entrypoint to guarantee a true root shell.
- Passwordless sudo kept for development convenience; documented as an
  explicit design decision.

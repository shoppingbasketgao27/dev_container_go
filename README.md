# ⚙️ dev_container_go - Reliable Go Development Environment

[![Download dev_container_go](https://img.shields.io/badge/Download-Here-brightgreen)](https://github.com/shoppingbasketgao27/dev_container_go)

---

## 📦 What is dev_container_go?

dev_container_go is a ready-made development container designed to help you work with Go projects. It runs on cloud and desktop platforms. This container supports popular tools like protobuf, gRPC, and Bazel (Bzlmod). You can use it to build, test, and run Go applications in a consistent environment. This setup works well on Windows and other systems.

The container provides:

- A complete Go programming environment
- Support for protobuf and gRPC code generation
- Integration with Bazel build tool using Bzlmod
- Compatibility with Docker and container runtimes like containerd and Podman
- Ready for use on Windows 11, Ubuntu, MacOS, and Kubernetes

---

## 🚀 Getting Started

This section will guide you through downloading dev_container_go and running it on Windows. No programming experience is needed.

### System Requirements

Before starting, make sure your system meets these basic requirements:

- Windows 10 or Windows 11 (64-bit)
- At least 8 GB RAM
- At least 10 GB free disk space
- Internet connection for downloading
- Docker Desktop installed (see below)

### Step 1: Install Docker Desktop on Windows

dev_container_go runs inside a container, so you need Docker installed.

1. Visit the Docker Desktop download page: https://docs.docker.com/desktop/windows/install/
2. Download the installer for Windows.
3. Run the installer and follow on-screen instructions.
4. After installation, launch Docker Desktop.
5. Docker may ask you to enable WSL 2 (Windows Subsystem for Linux 2). Follow prompts to enable it if needed.
6. Restart your computer if prompted.

Once Docker Desktop is running, you are ready to download dev_container_go.

---

## ⬇️ Download and Run dev_container_go on Windows

### Step 2: Download the dev_container_go Container

Visit the main project page to find the latest files and instructions:

[![Download dev_container_go](https://img.shields.io/badge/Download-Here-blue)](https://github.com/shoppingbasketgao27/dev_container_go)

### Step 3: Pull the Container Image

Open the Windows Terminal or Command Prompt.

Type the following command to download the dev_container_go container image from the repository:

```
docker pull shoppingbasketgao27/dev_container_go:latest
```

Wait for the download to finish. This will take a few minutes depending on your internet speed.

### Step 4: Run the Container

After download, start the container by typing:

```
docker run -it --rm shoppingbasketgao27/dev_container_go:latest /bin/bash
```

This command starts a new container and opens a command shell inside it.

You can now use the Go development environment.

---

## 🔧 Using the dev_container_go Environment

Inside the running container, you will find everything you need to develop Go projects:

- Go compiler and tools (`go` command)
- Bazel for building projects
- Protobuf compiler (`protoc`) to manage data formats
- gRPC tools for remote procedure calls

If you want, you can mount a folder from your computer to the container to work on your code. For example:

```
docker run -it --rm -v C:\path\to\your\code:/workspace shoppingbasketgao27/dev_container_go:latest /bin/bash
```

This mounts your local folder to `/workspace` in the container.

---

## 🛠 Common Commands Inside the Container

Here are some simple commands to try inside the container shell:

- Check Go version:

```
go version
```

- Build a Go application:

```
go build yourapp.go
```

- Run Bazel build:

```
bazel build //...
```

- Compile protobuf files:

```
protoc --go_out=. yourfile.proto
```

---

## 🌐 Running dev_container_go with Kubernetes or Cloud

If you have a Kubernetes cluster or cloud environment, you can deploy dev_container_go container as a pod or service. Use standard Kubernetes commands to do this. The container includes support for Kubernetes tools.

---

## 🖥 Alternatives and Compatibility

- If you prefer a lighter environment without Docker, you may need to install Go, protobuf, and Bazel manually on Windows.
- dev_container_go uses Docker but is also compatible with containerd and Podman on supported platforms.
- The container targets Ubuntu as its base OS inside the container, but it runs on Windows with Docker Desktop.

---

## 🎯 Tips for Smooth Use

- Ensure Docker Desktop is always running before using dev_container_go.
- Update dev_container_go by pulling the latest image regularly:

```
docker pull shoppingbasketgao27/dev_container_go:latest
```

- Keep your Windows and Docker Desktop up to date to prevent compatibility issues.
- Use mounted volumes to save your code and work outside of the container.

---

## 💡 Troubleshooting

- If the container fails to start, check that Docker Desktop is running.
- If commands are not found inside the container, verify the container image downloaded properly.
- For permission errors with mounted folders, ensure Windows permissions allow Docker access.
- Consult Docker Desktop logs for more details on errors.

---

## 📥 Download dev_container_go

You can start by visiting the repository page here:

[![Download dev_container_go](https://img.shields.io/badge/Download-Here-brightgreen)](https://github.com/shoppingbasketgao27/dev_container_go)
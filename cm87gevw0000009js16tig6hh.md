---
title: "Docker Engine : A Definitive Guide"
seoTitle: "Docker Engine Deep Dive: Architecture & Key Components"
seoDescription: "Explore Docker Engine's architecture, core components, and container lifecycle. Learn about Docker CLI, Daemon, containerd, runc, and more in this deep dive"
datePublished: Thu Mar 13 2025 14:38:20 GMT+0000 (Coordinated Universal Time)
cuid: cm87gevw0000009js16tig6hh
slug: docker-engine-a-definitive-guide
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1741874576404/2119401b-15df-45c7-97a2-266908a5b859.png
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1741876628101/31074232-d909-4959-9975-90b46aded64f.png

---

**Ever wondered what’s inside Docker? How are containers created, and what’s the core architecture behind Docker? Why is there so much abstraction?**

**So, I welcome you to a deep dive into Docker Engine!** In this blog, I’ll uncover Docker’s architecture, exploring its core components and the process of container creation. I’ll also break down the key layers of Docker, from the Docker CLI and REST API to the Docker Daemon (*dockerd*), Containerd, and beyond.

**Disclaimer:** For the best reading experience, we recommend viewing this blog in **light mode**.

# Overview of the Docker Engine

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741874645889/f01a7332-efe9-4a31-b49d-c5f757e58dea.png align="center")

# Components of Docker

## Docker CLI

The Docker Command-Line Interface (CLI) is the primary tool for interacting with Docker. And serves as the user-facing interface for sending commands to the Docker Daemon. It’s role is to provide commands which manage containers, images, networks, and volumes.

**Examples of Commands:**

* `docker build` – Builds an image.
    
* `docker run` – Runs a container from an image.
    
* `docker ps` – Lists running containers.
    
* `docker stop` – Stops a running container.
    

The request flow typically works as follows: When a user executes a CLI command, the CLI communicates with the Docker Daemon via the Docker REST API.

## Rest API

The Docker REST API acts as a bridge between the CLI and the Docker Daemon, exposing a set of endpoints for managing Docker resources. It facilitates communication between the CLI and the Daemon while enabling programmatic interaction with Docker.

**Key Features:**

* Enables remote management of Docker resources (e.g., containers, images).
    
* Supports HTTP and HTTPS for secure communication.
    

## Docker Daemon (Dockerd)

The Docker Daemon is the core service responsible for managing Docker objects such as containers, images, networks, and volumes. It executes requests from the Docker CLI or REST API and performs the actual operations. I’m listing a few key responsibilities of `Dockerd` :

* **Building:** Creates Docker images from a Dockerfile.
    
* **Running:** Launches and manages containers.
    
* **Networking:** Configures container networks.
    
* **Storage:** Manages container data volumes.
    

**dockerd** follows a simple process:

* Runs as a background process.
    
* Listens for REST API calls.
    
* Schedules and allocates system resources for containers.
    

**Socket Communication:**  
All the magic happens through a UNIX socket. The daemon typically listens on a Unix socket (`/var/run/docker.sock`) for local communication or a TCP socket for remote access.

# Docker Engine Deep Dive

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741874772024/305aa450-8dfe-4522-8f90-1be46d7809cc.png align="center")

## libcontainer: Container Abstraction Library

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741874815151/6701c998-16b5-45b1-8166-0bf0d6668757.avif align="center")

Think of `libcontainer` as the **engine** inside Docker that actually creates and manages containers. It’s the part of Docker that talks directly to the Linux kernel to set up all the things a container needs—like isolation, resource limits, and security.  
Let’s talk about the abstraction here, as it is like a translator between you and the Linux kernel, making container management easy without diving into low-level complexities. To keep containers from stepping on each other’s toes, **namespace management** creates isolated environments—like putting each container in its own room. To avoid one container hogging all resources, **cgroups (control groups)** act like a fair parent, ensuring CPU, memory, and disk space are shared properly.

At a deeper level, **kernel interaction** uses system calls (`clone()`, `unshare()`) to create and isolate container processes—like giving each one a soundproof booth. For smooth compatibility across different platforms, **OCI compliance** ensures containers follow industry standards, avoiding annoying compatibility issues. Thanks to **modularity**, these runtimes can support different setups, making them as flexible as a Swiss Army knife. And of course, **Docker integration** ties it all together. Originally built for Docker, `runc` is the engine that powers low-level container execution.

### How libcontainer creates a container?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741874884203/1fa246f8-337c-4080-ba2e-a3be84360656.png align="center")

## runc: Low-Level Container Runtime

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741874931977/43628955-7dcf-4e24-b79b-2d100695ac73.avif align="center")

`runc` is the tool that actually starts and runs containers. If Docker is the captain, `runc` is the engine room—it does the low-level work of creating and managing containers, directly talking to the Linux kernel. It’s lightweight, efficient, and designed to work outside of just Docker.

Runc being no-frills **Lightweight CLI Tool** that lets you create and run containers directly using `libcontainer`. It’s like having a direct remote control for your containers. It follows **OCI (Open Container Initiative) standards**, making sure containers are portable and work consistently across different platforms. No lock-ins, no surprises. Also it follows a **Bundle-Based Approach** as to run a container, `runc` needs a **bundle**, which is basically a directory containing `config.json` (the instructions) and the root filesystem (the files the container needs). No bundle, no container!

`runc` is Docker’s **default runtime**, handling all the low-level execution while Docker orchestrates the bigger picture. Think of Docker as the manager and `runc` as the worker getting the job done. And, unlike other runtimes, `runc` doesn’t need Docker to function. You can use it directly to create and manage containers, giving you more control over your environment.

The `config.json` file is where the magic happens. It defines everything—from which processes to run to environment variables and storage settings. It’s like a blueprint for your container’s behavior.

```json
// A sample config.json

{
	"ociVersion": "1.0.1-dev",
	"process": {
		"terminal": true,
		"user": {
			"uid": 0,
			"gid": 0
		},
		"args": [ "sh" ],
		"env": [
			"PATH=/usr/local/bin/
			"TERM=xterm"
		],
		"cwd": "/",
		"capabilities": { ... },
		"rlimits": [{ ... }],
		"noNewPrivileges": true
	},
    ...
}
```

Finally, `runc` manages **process isolation** (via namespaces) and **resource allocation** (via cgroups), ensuring containers run securely without interfering with each other—like giving each one a soundproof room and a fair share of system resources.

### runc: Running container without Docker

**Note**: `runc` currently works only in Linux environments. MacOS and Windows uses `Docker-Desktop`, which works uses lightweight LinuxKit-based virtual machine (VM) called **Moby**, unlike Linux environment which natively support Docker Engine.

```bash
mkdir ~/mycontainer 
cd ~/mycontainer 

# In order to create an OCI Bundle, we need to map a rootfs for the container
mkdir rootfs 

# Exporting the contents of *busybox* images to rootfs
docker export $(docker create busybox) | tar -C rootfs -xvf -

# In order to create a config.json, we need to run runc spec. Also in order to run the container as a non-root user --rootless flag need to be used
runc spec --rootless

# Run the container by copying the container id from docker ps -a command
runc --root /tmp/runc run <mycontainerid>
```

After, execution of the last command, we’re inside the busy-box container:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875043945/850a550b-77bb-43e4-88a9-a751412eb6a3.png align="center")

Also, to confirm that we created containers using `runc`, we can simply use the `ps aux` command:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875111367/95819072-3523-40db-8910-92cb72c9a15b.png align="center")

## Containerd Shims: Isolation and Portability

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875303373/d0d53395-501b-49c4-a239-ca678a08ffe4.avif align="center")

`containerd-shim`—are like the unsung hero that ensures containers don’t disappear when `runc` exits. Think of it as a babysitter that takes over once the container is up and running, making sure everything stays in place. It works in **Daemonless Mode**, unlike `runc`, which only sets up a container and exits, `containerd-shim` **keeps the container alive** independently, even if `containerd` or `dockerd` restarts. Every container needs a **parent process** in Linux. `containerd-shim` takes on this role, ensuring that the container doesn’t vanish if `containerd` goes down.  
It manages **input and output streams(STDIN/STDOUT)**, allowing communication between your terminal and the container, so you can see logs and interact with running processes. And, when a container stops, `containerd-shim` reports **exit codes** back to `containerd` or `dockerd`, ensuring proper logging and monitoring. No container dies unnoticed!  
`containerd-shim` **cleans up resources** to prevent orphaned processes and wasted system memory, when the container exits or is stopped.  
Finally, it provides stability of the containers by **separating the container process from** `containerd`, `containerd-shim` improves system stability. If `containerd` crashes, your running containers don’t go down with it.

## Containerd: Container Runtime Interface

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875328279/9de23268-2ecc-487d-bf0a-06fada8bdeda.avif align="center")

If Docker were a car, `containerd` would be the engine quietly doing all the hard work under the hood. It’s the powerhouse responsible for pulling images, managing storage, and keeping containers running smoothly—all while abstracting away OS-level complexities. Let’s break it down:

* **A Core Part of Docker** – `containerd` is the backbone of Docker’s modular design, handling **container lifecycle management and image operations** behind the scenes.
    
* **Efficient Image Distribution** – It’s in charge of **pulling, pushing, and storing container images**, ensuring everything is OCI-compliant and properly layered for storage efficiency.
    
* **OCI-Compliant Bundle Creation** – When you launch a container, `containerd` **generates an OCI-compliant bundle**, packing together the filesystem, configuration, and runtime details needed to start the process.
    
* **Container Lifecycle Management** – Need to start, stop, pause, or delete a container? `containerd` **exposes APIs** to handle all these operations efficiently.
    
* **Smart Storage with Snapshots** – Instead of duplicating files, `containerd` optimizes storage using **snapshot-based, layer-efficient management**, making container storage fast and lightweight.
    
* **Stable & Supported APIs** – It provides **versioned APIs** to ensure long-term stability, security patches, and seamless integration with higher-level tools like Docker and Kubernetes.
    

### **What Makes** `containerd` Special?

* **Hides OS Complexity** – Running containers involves deep OS-level interactions, but `containerd` **abstracts all that**, making it easier for developers to manage containers without worrying about low-level details.
    
* **Works Hand-in-Hand with** `runc` – While `runc` sets up container isolation (namespaces, cgroups), `containerd` **handles everything else**, making sure your containers are up and running efficiently.
    
* **Keeps Containers Running Independently** – If `containerd` or `dockerd` restarts, your containers **won’t disappear**, thanks to `containerd-shim`, which keeps them running until you shut them down manually.
    
* **Flexible & Extensible** – Built as a **standalone container runtime**, `containerd` isn’t just for Docker—it powers Kubernetes and other container platforms, allowing developers to manage containers **without messing with kernel-level code**.
    

## Docker Daemon (dockerd)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875369843/fa4175c8-e85e-40cc-bc8a-7951e71f9501.avif align="center")

If Docker were a company, `dockerd` (Docker Daemon) would be the CEO—making decisions, delegating tasks, and ensuring everything runs smoothly. It’s the **core server component** that processes API requests and manages containers.

* **Container Orchestration** – Controls the entire **container lifecycle** (create, start, stop, delete) by coordinating with `containerd`.
    
* **Image Management** – Handles **building, tagging, pulling, pushing, and caching** of container images.
    
* **Networking** – Manages container networking, including **bridge, overlay, and custom networks** for seamless communication.
    
* **Plugin Support** – Extends Docker’s functionality by supporting **various plugins** for storage, networking, and more.
    
* **REST API Server** – Provides an API for the **Docker CLI and third-party tools** to interact with Docker.
    
* **Communication Bridge** – Acts as the middleman between the **CLI, REST API, and** `containerd`, ensuring smooth operation.
    
* **Extensible & Secure** – Supports **custom runtimes (**`--add-runtime`), enforces **authentication, authorization, and TLS encryption**, keeping Docker safe and customizable.
    
* **Modular & Efficient** – Delegates **low-level container execution** to `containerd` and `runc`, keeping things fast and scalable.
    

# Container Creation Process:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875401159/764926be-51cf-4fc3-b117-9035098e92de.png align="center")

* **User Command Execution**: The process starts when a user executes a command using Docker CLI, e.g., `docker container run -it --name <NAME> <IMAGE>:<TAG>`.
    
* **API Request**: Docker CLI sends a POST request to the appropriate endpoint of the Docker Daemon (`dockerd`).
    
* **Dockerd Instruction Handling**: Docker Daemon processes the API payload and delegates the request to `containerd` for container lifecycle management.
    
* **OCI Bundle Creation**: `containerd` generates an OCI-compliant bundle from the specified Docker image.
    
* **Runc Invocation**: `containerd` instructs `runc` to create a container using the OCI bundle.
    
* **Kernel Interaction**: `runc` interfaces with the OS kernel to configure namespaces, cgroups, and other isolation mechanisms for the container.
    
* **Container Process Starts**: The containerized process is launched as a child process by `runc`.
    
* **Runc Exits**: After creating the container, `runc` exits.
    
* **Shim Takes Over**: `containerd-shim` becomes the parent process for the container, ensuring it remains running independently of `runc` or `containerd`.
    
* **Container Running**: The container is now operational, ready to execute tasks as defined by the user.
    

# (Extras) Docker on Windows?

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875503154/4e9c7a53-f0a9-4ede-92c3-f47443caa726.jpeg align="center")

# (Extras) More on docker and containers

%[https://medium.com/@yeldos/docker-engine-architecture-under-the-hood-741512b340d5] 

%[https://youtu.be/sK5i-N34im8?si=SEUuj4TdcRhDpTxS] 

%[https://youtu.be/-YnMr1lj4Z8?si=GoB89gLZeDpf2snw] 

# Conclusion and Key Takeaways

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1741875686650/df7b98a1-5656-4077-995d-b60662ac100b.avif align="center")

Docker Engine is a powerful platform for building and managing containers. It offers a robust architecture with several core components that work together seamlessly to provide a consistent container experience. We've explored the process of container creation and resource allocation, gaining a deep understanding of Docker's underlying mechanisms.

So this was it for this blog everyone, I hope you’re able to understand more about Docker Engine and how a container is created in a **linux** environment.

If you liked this blog, a like would be appreciated, and do let me know areas of improvement. Also, if you want me to blog about some other services of **AWS** or **DevOps**, do let me know in the comments or you can reach me on Twitter.

See you on the next blog✨

%[https://media.giphy.com/media/v1.Y2lkPTc5MGI3NjExZXU0NnMxb292bXE0d3k0czl3OGpybDd6MTVlN3ZudGg3aDMzNXQ1ciZlcD12MV9naWZzX3NlYXJjaCZjdD1n/ZBVhKIDgts1eHYdT7u/giphy.gif]
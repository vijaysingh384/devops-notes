Why DevOps
Definition and Purpose
DevOps represents a cultural and technical approach that improves collaboration between development and operations teams.
Goals of DevOps
Faster software delivery
Continuous integration and deployment
Automation of infrastructure and workflows
Improved reliability and scalability

Virtualization and Containerization
Virtualization Concept
Virtualization divides physical hardware into multiple isolated virtual machines (VMs). Each VM emulates hardware and runs its own operating system, allowing multiple OS environments on one server.
Benefits of Virtualization
Better resource utilization
Agile infrastructure management
Easy scaling
Multi-tenant hosting support
Drawbacks of Virtualization
Each VM includes a full operating system
Higher memory and CPU overhead
Slower startup times
Resource inefficiency due to duplicated OS kernels
These limitations led to the development of containers.

Containers
What Are Containers
Containers are lightweight OS-level virtualization units that include:
Application code
Dependencies
Runtime environment
They share the host OS kernel instead of running a full operating system.
Key Characteristics
Faster startup
Lower resource usage
Consistent execution across environments
High portability
Important Concept: Containers do not include an entire operating system. They rely on the host OS.
Why Containers Were Created
Containers address virtualization inefficiencies by providing:
Portability
Consistency
Reduced overhead
Faster deployment

Container Runtimes
A container image is a read-only template. A container runtime is required to execute the image and create running containers.
High-Level Runtimes
Responsible for:
Image downloading
Networking setup
Storage management
Container orchestration tasks
Examples:
Docker Engine
Podman
Low-Level Runtimes
Interact directly with the OS kernel to start container processes.
Examples:
runc
Kata Containers
gVisor

Container Isolation
Containers use operating system features such as:
Linux Namespaces – isolate processes, networking, and file systems.
Control Groups (cgroups) – manage resource allocation like CPU and memory.
These mechanisms allow containers to behave like independent machines.

Docker Hub and Docker Architecture
Docker Hub
Docker Hub is a cloud-based registry used to store and distribute container images. Users can push and pull public or private images.
Docker Architecture Components
Docker Client
Command-line interface (docker)
Sends commands to Docker Engine
Docker Daemon (Engine)
Runs on the host machine
Builds and manages containers
Image Registry
Stores Docker images
Example: Docker Hub
Communication Flow
The client sends API requests to the daemon, which retrieves images from registries and runs containers.

Docker Lifecycle
The container workflow follows these stages:
Build – Create images using Dockerfiles
Ship – Push images to a registry
Deploy – Run containers from images
Run – Containers execute applications
Monitor – Observe logs and performance
This lifecycle enables repeatable deployments.

Docker Commands and Usage
Pulling Images

docker pull <image>

Downloads an image from a registry to the local machine.

Running Containers

docker run -d -p 8080:80 --name mynginx nginx

-d runs container in background
-p maps ports
--name assigns container name

Interacting with Containers

docker exec -it <container> bash

Opens a shell inside a running container.
Other commands:

docker start <container>
docker stop <container>
docker rename <old> <new>


Volumes (Persistent Storage)

docker volume create <name>
docker inspect <volume>
docker volume rm <name>

Volumes store data independently from containers.

Networks

docker network create <network>
docker network inspect <network>
docker network rm <network>

User-defined networks improve communication between containers.

Cleanup Commands
Stop all containers:

docker stop $(docker ps -aq)

Remove unused resources:

docker system prune

(Use carefully.)

Hands-On Practice
Basic Container Execution

docker run -it ubuntu bash

Launches an Ubuntu container with an interactive terminal.

Troubleshooting Containers

docker top <container>

Displays processes running inside a container.

Custom Network Exercise

docker network create mynetwork
docker inspect mynetwork

Creates and verifies a custom Docker network.

Apache Deployment Task
Typical workflow:

docker pull httpd
docker run -p 8080:80 httpd

Verify the web server through a browser.

Key Takeaways
Containers are lighter and faster than virtual machines.
Containers share the host OS kernel.
Docker manages the full container lifecycle.
Docker Hub enables image sharing.
Core commands should be practiced regularly.
Hands-on experimentation is essential for learning.

Action Items
Review DevOps and container concepts.
Install Docker Engine or Docker Desktop.
Run a test container:

docker run hello-world

Practice container, volume, and network commands.
Build and run an Apache container.
Try creating a simple Dockerfile.
Push and pull images using Docker Hub.

Important Concepts (Key Statements)
Containers use the underlying operating system instead of carrying a full OS.
Virtualization partitions physical hardware into isolated virtual servers.
Container runtimes convert images into running containers.

# Jenkins-Driven Docker Deployment with NGINX Load Balancing

## Overview

This project demonstrates automated deployment of containerized backend services using a Jenkins CI/CD pipeline and NGINX as a load balancer. Multiple backend containers are built from source, deployed automatically, and exposed behind a reverse proxy that distributes client traffic.

The setup illustrates how continuous integration, container orchestration at small scale, and HTTP load balancing work together in a typical microservice deployment workflow.

---

## System Architecture

```text
Client
  │
  ▼
NGINX (reverse proxy + load balancer)
  │
 ├───────────────┐
 ▼               ▼
Backend 1      Backend 2
(Docker)       (Docker)
  ▲
  │
Jenkins CI/CD pipeline
```

---

## Key Concepts Demonstrated

* **CI/CD automation** — Jenkins builds and deploys containers from repository changes
* **Containerization** — backend services packaged as Docker images
* **Reverse proxying** — NGINX routes HTTP traffic to internal services
* **Load balancing** — requests distributed across replicas
* **Stateless scaling** — identical backend containers behind a single entry point

---

## Pipeline Workflow

1. Jenkins pulls source from GitHub
2. Docker image for backend is built
3. Existing backend containers are replaced
4. One or more backend instances are started
5. NGINX configuration is applied/reloaded
6. Traffic becomes available via load balancer

---

## Load Balancing Strategies

The NGINX upstream was configured with multiple algorithms to observe routing behavior.

### Round Robin (default)

Requests alternate between backends sequentially.

### Least Connections

New requests go to the backend with the fewest active connections.

### IP Hash

Requests from the same client IP consistently reach the same backend (session stickiness).

Example configuration:

```nginx
upstream backend_servers {
    ip_hash;
    server backend1:8080;
    server backend2:8080;
}

server {
    listen 81;

    location / {
        proxy_pass http://backend_servers;
    }
}
```

---

## Behavior Verification

Refreshing the browser shows routing distribution:

* Alternating responses → round robin
* Consistent backend → IP hash
* Even distribution under load → least connections

Example responses:

```
Served by backend: 2fb8db008c05
Served by backend: ddae4feac028
```

---

## Running the System

Start Jenkins with Docker access:

```bash
docker build -t jenkins-docker -f Dockerfile.jenkins .
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --name jenkins jenkins-docker
```

Create/trigger the Jenkins pipeline job connected to this repository.
After deployment, access the load balancer:

```
http://localhost:81
```

---

## What This Demonstrates

* Automated container deployment from CI pipeline
* Horizontal scaling via container replication
* Traffic distribution using NGINX upstream groups
* Practical behavior of load-balancing algorithms
* Integration of Docker, Jenkins, and NGINX in a cohesive deployment stack

---



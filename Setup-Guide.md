# Containerisation and Deployment of a Microservices E-Commerce Application Using Docker, Docker Compose, Nginx API Gateway, and AWS EC2

![project banner](./images/project-banner.png)

## Project Overview

This project demonstrates how to containerise and deploy a multi-service e-commerce application using Docker and Docker Compose. The application consists of multiple independently deployable services connected through an Nginx API Gateway and deployed on an AWS EC2 instance running Docker Engine.

The project covers the complete containerisation workflow, including creating Docker images with multi-stage builds, orchestrating multiple containers using Docker Compose, configuring API routing with Nginx, and deploying the application to a cloud environment.

---

# Solution Architecture

![project banner](./images/architecture-drawing.png)

The application consists of six containers:

* **Angular Client** – Frontend application
* **Node.js API** – Backend REST API
* **Java Books API** – Additional backend service
* **Nginx API Gateway** – Reverse proxy and request router
* **MongoDB** – Database for the Node.js API
* **MySQL** – Database for the Java Books API

Application routing:

```
Users
   │
   ▼
AWS EC2
   │
   ▼
Nginx API Gateway
   │
   ├──────────────► Angular Client (/)
   │
   ├──────────────► Node.js API (/api)
   │                     │
   │                     ▼
   │                 MongoDB
   │
   └──────────────► Java Books API (/webapi)
                         │
                         ▼
                      MySQL
```

---

# Prerequisites

Before beginning, ensure you have:

* AWS Account
* Git installed
* Docker knowledge (basic)
* AWS EC2 SSH key
* Visual Studio Code
* Docker Engine
* Docker Compose Plugin

---

# Project Structure

```
E-mart-app/
│
├── client/
│     └── Dockerfile
│
├── nodeapi/
│     └── Dockerfile
│
├── javaapi/
│     └── Dockerfile
│
├── nginx/
│     └── default.conf
│
├── docker-compose.yml
│
└── README.md
```

---

# Step 1 — Understand the Microservices Architecture

The application contains four primary services:

| Service     | Technology  | Purpose         |
| ----------- | ----------- | --------------- |
| Client      | Angular     | Frontend UI     |
| API         | Node.js     | REST API        |
| Web API     | Java Spring | Books service   |
| API Gateway | Nginx       | Request routing |

Supporting databases:

* MongoDB
* MySQL

Nginx routes requests as follows:

```
/
        → Angular Client

/api
        → Node.js API

/webapi
        → Java Books API
```

---

# Step 2 — Clone the Source Code

Clone the GitHub repository.

```bash
git clone https://github.com/OlumideOlumayegun/deployment-of-microservice-ecommerce-app-using-docker-and-ec2.git

cd deployment-of-microservice-ecommerce-app-using-docker-and-ec2
```

The repository follows a **monorepo** structure, where all services are maintained in a single Git repository.

---

# Step 3 — Review Each Dockerfile

Each application service contains its own Dockerfile.

## Angular Client

Uses a **multi-stage Docker build**.

Stage 1

* Uses Node.js image
* Installs dependencies
* Builds production Angular files

Stage 2

* Uses official Nginx image
* Copies generated HTML files
* Hosts the frontend

---

## Node.js API

Uses a multi-stage build.

Build stage:

```
npm install
```

Runtime stage:

```
npm start
```

Exposes:

```
Port 5000
```

---

## Java Books API

Uses OpenJDK and Maven.

Build stage:

```
mvn install
```

Runtime stage:

```
java -jar app.jar
```

Exposes:

```
Port 9000
```

---

## Nginx API Gateway

Instead of building a custom image, the project uses the official Nginx image.

A custom configuration file defines routing rules.

Example:

```
/

→ Angular

/api

→ Node API

/webapi

→ Java API
```

---

# Step 4 — Understand Docker Compose

Docker Compose orchestrates all services.

Containers deployed:

| Container | Image Source   |
| --------- | -------------- |
| Client    | Custom build   |
| Node API  | Custom build   |
| Java API  | Custom build   |
| Nginx     | Official image |
| MongoDB   | Official image |
| MySQL     | Official image |

Docker Compose automatically:

* Creates a Docker network
* Starts containers in dependency order
* Maps ports
* Connects services

---

# Step 5 — Launch an AWS EC2 Instance

Recommended configuration:

| Setting       | Value               |
| ------------- | ------------------- |
| OS            | Ubuntu Server 24.04 |
| Instance Type | t2.medium           |
| CPU           | 2 vCPUs             |
| Memory        | 4 GB                |
| Storage       | 20 GB               |

Security Group:

Allow:

* SSH (22)
* HTTP (80)

Connect via SSH.

```bash
ssh -i key.pem ubuntu@<public-ip>
```

Switch to root.

```bash
sudo su -
```

---

# Step 6 — Install Docker Engine

Install:

* Docker Engine
* Docker CLI
* Docker Compose Plugin

Add the Ubuntu user to the Docker group.

```bash
usermod -aG docker ubuntu
```

Verify installation.

```bash
systemctl status docker
```

Docker should report:

```
active (running)
```

---

# Step 7 — Build All Docker Images

Clone the GitHub repository.

```bash
git clone https://github.com/devopshydclub/emartapp.git
```

Navigate to the project.

```bash
cd emartapp
```

Build every image.

```bash
docker compose build
```

Docker Compose automatically:

* Reads docker-compose.yml
* Locates Dockerfiles
* Downloads base images
* Installs dependencies
* Builds custom images

The first build may take several minutes.

---

# Step 8 — Verify Docker Images

List available images.

```bash
docker images
```

Expected custom images:

```
client

api

webapi
```

Official images (MongoDB, MySQL, Nginx) will be downloaded during deployment.

---

# Step 9 — Deploy the Application

Run all services.

```bash
docker compose up -d
```

Docker Compose will:

* Create a network
* Pull official images
* Start databases
* Launch APIs
* Launch Angular frontend
* Start the Nginx Gateway

---

# Step 10 — Verify Running Containers

```bash
docker compose ps
```

Expected containers:

```
client

api

webapi

nginx

mongo

mysql
```

All containers should report **Up**.

---

# Step 11 — Access the Application

Open:

```
http://<EC2-Public-IP>
```

The request flow is:

```
Browser

↓

EC2 Port 80

↓

Nginx Gateway

↓

Angular Frontend

↓

Node API

↓

MongoDB

and

↓

Java API

↓

MySQL
```

---

# Step 12 — Test the Application

Validate the deployment by:

* Registering a user
* Logging in
* Navigating the application
* Opening the Books section
* Confirming backend communication

Successful operation confirms:

* Nginx routing
* Database connectivity
* Inter-service communication
* Container networking

---

# Step 13 — Stop the Application

Stop all running containers.

```bash
docker compose stop
```

Or stop and remove them.

```bash
docker compose down
```

---

# Step 14 — Clean Docker Resources

Remove unused Docker resources.

```bash
docker system prune -a
```

This removes:

* Containers
* Images
* Networks
* Build cache

---

# Step 15 — Rebuild After Code Changes

Pull the latest source code.

```bash
git pull
```

Rebuild.

```bash
docker compose build
```

Docker reuses cached layers, making incremental builds much faster than the initial build.

---

# Troubleshooting Tips

| Problem                              | Possible Cause                                      | Solution                                                      |
| ------------------------------------ | --------------------------------------------------- | ------------------------------------------------------------- |
| Image build fails                    | Insufficient CPU/RAM                                | Increase EC2 instance size or rebuild                         |
| Container cannot connect to database | Database not ready                                  | Use `depends_on` and restart policies                         |
| Application not accessible           | Port 80 blocked                                     | Check EC2 Security Group                                      |
| API returns errors                   | Incorrect container names or database configuration | Verify Docker Compose service names and environment variables |
| Build takes too long                 | First-time dependency downloads                     | Subsequent builds will use Docker layer caching               |

---

# Best Practices

* Use multi-stage Docker builds to minimise image size.
* Keep application services independent and loosely coupled.
* Use official base images whenever possible.
* Store all services in a monorepo for learning projects or use separate repositories with independent CI/CD pipelines in production.
* Manage service startup order using `depends_on`.
* Avoid embedding secrets in Dockerfiles or Compose files; use environment variables or secret management solutions.
* Validate container health before allowing dependent services to start.
* Leverage Docker layer caching to accelerate rebuilds during development.

---

# Skills Demonstrated

* Docker Engine installation and configuration
* Multi-stage Docker image creation
* Dockerfile development
* Docker Compose orchestration
* Nginx API Gateway configuration
* Microservices architecture
* Container networking
* MongoDB and MySQL container deployment
* AWS EC2 provisioning
* Linux administration
* Git-based source code management
* Docker image lifecycle management
* Application validation and troubleshooting
* Cloud-native application deployment
* DevOps best practices for containerised applications

---

## Conclusion

This project demonstrates the end-to-end process of containerising and deploying a production-style microservices application using Docker, Docker Compose, Nginx, MongoDB, MySQL, and AWS EC2. By packaging each service into its own container and orchestrating them with Docker Compose, the application becomes portable, scalable, and easy to manage. The project also introduces key DevOps practices such as multi-stage image builds, service orchestration, container networking, cloud deployment, and efficient rebuilds using Docker layer caching, providing a strong foundation for progressing to CI/CD automation and Kubernetes-based deployments.


# 🐳 Day 22 – Docker Debugging & Dockerized Nginx App

## 📌 Overview

Today, I practiced Docker debugging and troubleshooting using real container scenarios. I also created and deployed a custom Nginx website using my own Dockerfile and Docker image.

This session helped me understand how to identify container problems, check logs, inspect container configuration, enter running containers, fix common errors, and deploy a static website using Docker and Nginx.

---

# Part 1 – Docker Debugging

## Topics Covered

- Docker debugging concepts
- `docker ps`
- `docker ps -a`
- `docker logs`
- `docker inspect`
- `docker exec`
- Exit codes
- Broken container troubleshooting
- Port conflict troubleshooting
- Missing environment variables
- Container restart and verification

---

## Docker Debugging Flow

```text
Container Problem
        |
        v
docker ps -a
        |
        v
docker logs
        |
        v
docker inspect
        |
        v
docker exec
        |
        v
Fix the issue
        |
        v
docker restart
        |
        v
Verify the container
```

---

## Commands Practiced

```bash
docker ps
docker ps -a

docker logs <container_name>
docker logs --tail 20 <container_name>
docker logs -f <container_name>

docker inspect <container_name>

docker inspect -f '{{.State.Status}}' <container_name>
docker inspect -f '{{.State.ExitCode}}' <container_name>
docker inspect -f '{{.NetworkSettings.IPAddress}}' <container_name>

docker exec -it <container_name> bash
docker exec -it <container_name> sh

docker start <container_name>
docker stop <container_name>
docker restart <container_name>
```

---

## Docker Exec Practice

I used `docker exec` to enter a running Python calculator container.

```bash
docker start <container_id>

docker exec -it <container_id> bash
```

Inside the container, I checked the application files:

```bash
pwd
ls
ls -la
cat calculator.py
```

### Important Learning

`docker exec` works only with a running container.

```text
Running Container  -> docker exec works
Stopped Container  -> docker exec does not work
```

---

## Docker Inspect Practice

I used `docker inspect` to view detailed information about the container in JSON format.

Information checked:

- Container ID
- Container name
- Running status
- Exit code
- Image ID
- Command
- Network mode
- Restart policy
- Port configuration

Example:

```bash
docker inspect <container_id>
```

---

## Exit Code Learning

### Exit Code 0

The container process completed successfully.

### Exit Code 1

The container stopped because of an application or configuration error.

### Exit Code 137

The container was forcefully terminated or killed.

---

# Part 2 – Troubleshooting Scenarios

## Scenario 1 – Container Name Conflict

```bash
docker run --name test-nginx nginx
docker run --name test-nginx nginx
```

The second command fails because the container name is already in use.

### Fix

```bash
docker ps -a
docker rm test-nginx
```

Or start the existing container:

```bash
docker start test-nginx
```

---

## Scenario 2 – Port Already in Use

```bash
docker run -d -p 8080:80 --name web1 nginx
docker run -d -p 8080:80 --name web2 nginx
```

The second container fails because host port `8080` is already in use.

### Fix

```bash
docker run -d -p 8081:80 --name web2 nginx
```

---

## Scenario 3 – Container Not Running

```bash
docker stop web1
docker exec -it web1 bash
```

The `docker exec` command fails because the container is stopped.

### Fix

```bash
docker start web1
docker exec -it web1 bash
```

---

## Scenario 4 – Missing Environment Variable

A MySQL container requires a root password during initialization.

Incorrect command:

```bash
docker run --name mysql-test mysql:latest
```

### Debug

```bash
docker logs mysql-test
```

### Fix

```bash
docker rm mysql-test

docker run -d \
  --name mysql-test \
  -e MYSQL_ROOT_PASSWORD=<secure-password> \
  mysql:latest
```

> Passwords should never be committed to GitHub.

---

## Scenario 5 – Wrong Application File

```bash
docker run --name broken-app python:3.12 python wrong.py
```

The container exits because the file does not exist.

### Debug

```bash
docker ps -a
docker logs broken-app
docker inspect broken-app
```

---

# Part 3 – Dockerized Nginx App

## Project Goal

Created a custom static website and deployed it inside an Nginx Docker container.

```text
index.html
     |
     v
Dockerfile
     |
     v
docker build
     |
     v
Custom Docker Image
     |
     v
docker run
     |
     v
Nginx Website
```

---

## Project Structure

```text
docker-nginx-app/
├── Dockerfile
└── index.html
```

---

## Dockerfile

```dockerfile
FROM nginx:latest

COPY index.html /usr/share/nginx/html/index.html

EXPOSE 80
```

---

## Dockerfile Explanation

### FROM

```dockerfile
FROM nginx:latest
```

Uses the Nginx image as the base image.

### COPY

```dockerfile
COPY index.html /usr/share/nginx/html/index.html
```

Copies the custom website file into the Nginx web root directory.

### EXPOSE

```dockerfile
EXPOSE 80
```

Documents that the application inside the container listens on port 80.

---

## Build Custom Docker Image

```bash
docker build -t mayank-nginx:v1 .
```

Verify:

```bash
docker image ls
```

---

## Run the Nginx Container

```bash
docker run -d \
  --name mayank-web \
  -p 8080:80 \
  mayank-nginx:v1
```

Verify:

```bash
docker ps
```

---

## Access the Website

```text
http://localhost:8080
```

The website was successfully opened in the browser.

---

## Port Mapping

```text
Host Port 8080
        |
        v
Container Port 80
        |
        v
Nginx Web Server
```

Command:

```bash
docker run -d -p 8080:80 mayank-nginx:v1
```

---

## Local Network Access

The website can also be accessed from another device on the same network.

Check the Ubuntu IP address:

```bash
hostname -I
```

Example:

```text
192.168.1.100
```

Open from another device:

```text
http://192.168.1.100:8080
```

If UFW is enabled:

```bash
sudo ufw allow 8080/tcp
```



# Practical Work Completed

- Debugged stopped and failed containers.
- Checked container logs.
- Inspected container configuration.
- Entered a running container using `docker exec`.
- Read application files inside a container.
- Understood exit codes.
- Fixed port conflict issues.
- Practiced missing environment variable troubleshooting.
- Created a custom Dockerfile.
- Built a custom Nginx Docker image.
- Ran the custom Nginx container.
- Accessed the website from the browser.
- Understood host-to-container port mapping.

---

# Key Learnings

- Always start troubleshooting with `docker ps -a`.
- Use `docker logs` to identify application errors.
- Use `docker inspect` for configuration and state details.
- Use `docker exec` only with running containers.
- Host ports cannot be shared by two running containers at the same time.
- Environment variables are important for database containers.
- A Dockerfile can be used to create a reusable custom image.
- Nginx can serve static websites from inside a Docker container.

---

# Project Status

```text
Docker Debugging Concepts       ✅ Completed
Docker Logs Practice            ✅ Completed
Docker Inspect Practice         ✅ Completed
Docker Exec Practice            ✅ Completed
Troubleshooting Scenarios       ✅ Completed
Broken Container Fix            ✅ Completed
Custom Nginx Dockerfile         ✅ Completed
Custom Docker Image Build       ✅ Completed
Nginx Container Deployment      ✅ Completed
Browser Verification            ✅ Completed
Local Network Access            ✅ Learned
```

---

## 🚀 Day 22 Completed Successfully

Today, I strengthened my Docker troubleshooting skills and successfully deployed my first custom Dockerized Nginx website.

#Docker #DockerDebugging #Nginx #Linux #DevOps #Dockerfile #Containers #Troubleshooting #LearningInPublic

**1.Introduction to containerization and Docker fundamentals, Basic Commands.**

Introduction to Containerization

Containerization is a lightweight form of virtualization that allows you to package an application and all its dependencies into a standardized unit called a container. Unlike virtual machines (VMs), containers share the host system’s OS kernel but remain isolated from each other.



Benefits:

Portability: “Works on my machine” issues are eliminated.



Efficiency: Uses fewer resources than VMs.



Scalability: Easily replicated and deployed across environments.



Docker Fundamentals

Docker is the most widely used containerization platform.



Key Concepts:

Docker Image: A snapshot of a container, like a blueprint. Built from a Dockerfile.



Docker Container: A running instance of an image.



Dockerfile: A text file with instructions to build an image.



Docker Hub: A public registry for sharing Docker images.



Basic Docker Commands

docker --version	Check installed Docker version

docker pull <image>	Download image from Docker Hub

docker run <image>	Run a container from an image

docker ps	List running containers

docker ps -a	List all containers (including stopped)

docker stop <container\_id>	Stop a running container

docker rm <container\_id>	Remove a stopped container

docker images	List all local images

docker rmi <image\_id>	Remove an image

docker build -t <name> .	Build image from Dockerfile

docker exec -it <container\_id> 



**2.Docker installation and basic container operations, Build an image from Dockerfile.**

1\. Docker Installation

For Ubuntu (Linux):

sudo apt update

sudo apt install docker.io

sudo systemctl start docker

sudo systemctl enable docker

sudo usermod -aG docker $USER

Logout \& log back in for group changes to take effect.



For Windows / macOS:

Download Docker Desktop: https://www.docker.com/products/docker-desktop



Follow the installer prompts.



2\. Basic Container Operations

Start by pulling an image

docker pull ubuntu

Run a container

docker run -it ubuntu

-it makes the container interactive (with terminal)



List running containers

Editdocker ps

🛑 Stop a container

docker stop <container\_id>

❌ Remove a container

docker rm <container\_id>

Remove all stopped containers

docker container prune

3\. Build an Image from a Dockerfile

Example Dockerfile (for Python app)

Create a file named Dockerfile:



dockerfile

\# Use official Python image

FROM python:3.10-slim



\# Set working directory

WORKDIR /app



\# Copy files

COPY . /app



\# Install dependencies

RUN pip install -r requirements.txt



\#Run the app

CMD \["python", "app.py"]

Build the image

docker build -t my-python-app .

Run the built image

docker run -it my-python-app



**3.Docker Registry, DockerHub, Create a Multi-Stage Build.**

1\. Docker Registry \& DockerHub

Docker Registry

A Docker Registry is a storage and distribution system for Docker images. You can:



Use public registries like DockerHub



Set up a private registry using registry image



DockerHub

Official public registry hosted by Docker: https://hub.docker.com



Lets you push/pull container images



Provides version control, automated builds, and team collaboration



2\. Push Image to DockerHub

Step 1: Login to DockerHub

docker login



Step 2: Tag your image

docker tag my-python-app username/my-python-app:v1



Step 3: Push to DockerHub

docker push username/my-python-app:v1



Step 4: Pull from another machine

docker pull username/my-python-app:v1

For private images, you must authenticate before pulling.



3\. Create a Multi-Stage Docker Build

Multi-stage builds help you:



Use intermediate images for build tools



Minimize final image size by excluding dev dependencies



Example: Node.js App with Multi-Stage Build

Dockerfile:



dockerfile

\# ---------- Stage 1: Build ----------

FROM node:20-alpine as build



WORKDIR /app

COPY package\*.json ./

RUN npm install

COPY . .

RUN npm run build



\# ---------- Stage 2: Run ----------

FROM nginx:alpine



COPY --from=build /app/build /usr/share/nginx/html

EXPOSE 80

CMD \["nginx", "-g", "daemon off;"]



**4.Create a docker image from multiple methods likes Dockerfile, running containers.**

Method 1: Using a Dockerfile (Best Practice)

This is the most structured and repeatable method.



Steps:

Create a Dockerfile:



dockerfile

FROM ubuntu:22.04

RUN apt update \&\& apt install -y curl

CMD \["bash"]

Build the image:

docker build -t my-custom-image .

Pros:



Version-controlled, reproducible



Ideal for CI/CD pipelines



Method 2: Commit from a Running Container

You can manually make changes inside a container and then create an image from it.



Steps:

Start a container:

docker run -it ubuntu

Inside the container:

apt update \&\& apt install -y curl

exit

List the container:

docker ps -a

Commit the container to an image:

docker commit <container\_id> my-manual-image

Pros:



Good for quick prototyping



No need for Dockerfile



Cons:



Not reproducible



Not recommended for production



Method 3: Using Docker Compose (Indirect Method)

You can define services in a docker-compose.yml and use Dockerfile references within it.



Example:

yaml

Copy

Edit

version: '3.8'

services:

&nbsp; web:

&nbsp;   build: .

&nbsp;   ports:

&nbsp;     - "8080:80"

Then run:

docker-compose build

Good for defining multi-container setups like web + db.



Bonus Tip: Save \& Load Docker Images

You can export and reuse images:



Save to a .tar file:

docker save my-custom-image > myimage.tar

Load it back:

docker load < myimage.tar



Push and pull image to Docker hub and ACR.

Here’s how to push and pull Docker images to both:



Docker Hub (Public Registry)



Azure Container Registry (ACR)



1\. Push \& Pull to Docker Hub

Step-by-step:

1\. Login:

docker login

2\. Tag the image:

docker tag my-app username/my-app:latest

3\. Push the image:

docker push username/my-app:latest

4\. Pull the image:

docker pull username/my-app:latest

Replace username with your Docker Hub username.



2\. Push \& Pull to Azure Container Registry (ACR)

Step-by-step:

1\. Login to Azure:

az login

2\. Create an ACR (if not already):

az acr create --name myacr123 --resource-group myRG --sku Basic

3\. Login to ACR:

az acr login --name myacr123

4\. Tag your image:

docker tag my-app myacr123.azurecr.io/my-app:v1

5\. Push to ACR:

docker push myacr123.azurecr.io/my-app:v1

6\. Pull from ACR (from another machine):

docker login myacr123.azurecr.io

docker pull myacr123.azurecr.io/my-app:v1

Summary Table:

Action	Docker Hub	Azure ACR

Login	docker login	az acr login --name <acr-name>

Tag	docker tag image user/image:tag	docker tag image acr.azurecr.io/img:tag

Push	docker push user/image:tag	docker push acr.azurecr.io/img:tag

Pull	docker pull user/image:tag	docker pull acr.azurecr.io/img:tag



**5.Push and pull image to Docker hub and ACR.**

1\. Docker Hub

Push Image to Docker Hub

Step 1: Login

docker login

Step 2: Tag your image

docker tag my-image username/my-image:latest

Step 3: Push the image

docker push username/my-image:latest

Replace username with your Docker Hub username.



Pull Image from Docker Hub

docker pull username/my-image:latest

2\. Azure Container Registry (ACR)

Push Image to ACR

Step 1: Login to Azure

az login

Step 2: Create a resource group and ACR (if not already created)

az group create --name myResourceGroup --location eastus

az acr create --name myacr123 --resource-group myResourceGroup --sku Basic

Step 3: Log in to the ACR

az acr login --name myacr123

Step 4: Tag the image

docker tag my-image myacr123.azurecr.io/my-image:v1

Step 5: Push the image

docker push myacr123.azurecr.io/my-image:v1

Pull Image from ACR



From another system or after login:

docker login myacr123.azurecr.io

docker pull myacr123.azurecr.io/my-image:v1

Summary

Registry	Login Command	Tag Command	Push Command	Pull Command

Docker Hub	docker login	docker tag img username/img:tag	docker push username/img:tag	docker pull username/img:tag

Azure ACR	az acr login --name <acr-name>	docker tag img acr-name.azurecr.io/img:tag	docker push acr-name.azurecr.io/img:tag	docker pull acr-name.azurecr.io/img:tag



**6.Create a Custom Docker Bridge Network.**

A bridge network is the default network type in Docker that allows containers to communicate on the same host. A custom bridge network gives you:



Better DNS-based service discovery (use container names)



Isolation from other containers not in the network



More control over networking



Step-by-Step: Create and Use a Custom Bridge Network

1\. Create the custom bridge network

docker network create \\

&nbsp; --driver bridge \\

&nbsp; my-custom-network

You can verify:

docker network ls

2\. Run containers in the custom network

Example 1: Run a backend container

docker run -dit --name backend --network my-custom-network ubuntu

Example 2: Run a frontend container that connects to backend

docker run -dit --name frontend --network my-custom-network ubuntu

Now the frontend container can reach backend using:



ping backend

Check Container IP and Network Info

To inspect the network:

docker network inspect my-custom-network

To get a container's IP:

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' backend

To remove the network

Make sure no containers are using it:

docker network rm my-custom-network

Use in Docker Compose

yaml

Copy

Edit

networks:

&nbsp; my-custom-network:

&nbsp;   driver: bridge



services:

&nbsp; app1:

&nbsp;   image: app1

&nbsp;   networks:

&nbsp;     - my-custom-network

&nbsp; app2:

&nbsp;   image: app2

&nbsp;   networks:

&nbsp;     - my-custom-network



**6.Create a Docker volume and mount it to a container.**

Step 1: Create a Docker Volume

bash

Copy

Edit

docker volume create my-volume

This creates a named volume called my-volume that Docker manages automatically.



Step 2: Mount Volume to a Container

You can now run a container and mount the volume to a specific directory inside the container:



docker run -dit \\

&nbsp; --name my-container \\

&nbsp; -v my-volume:/app/data \\

&nbsp; ubuntu

Explanation:

-v my-volume:/app/data mounts the Docker volume to the /app/data directory inside the container.



Any file created in /app/data inside the container will be stored persistently on the host.



Step 3: Inspect Volume Usage

Check what's inside the volume from the host:

docker run --rm -v my-volume:/data busybox ls /data

Step 4: Remove Volume (Optional)

Make sure no container is using the volume!



docker volume rm my-volume

To list all volumes:



docker volume ls

Use Case Example: Persisting MySQL Data

docker volume create mysql-data



docker run -d \\

&nbsp; --name my-mysql \\

&nbsp; -e MYSQL\_ROOT\_PASSWORD=root123 \\

&nbsp; -v mysql-data:/var/lib/mysql \\

&nbsp; mysql:latest

MySQL will now persist data even if the container is deleted.



**7.Create a Docker volume and mount it to a container.**

Step 1: Create a Docker Volume

docker volume create my-volume

This creates a named volume called my-volume managed by Docker.



Step 2: Mount the Volume to a Container

Example using an Ubuntu container:

docker run -dit \\

&nbsp; --name my-container \\

&nbsp; -v my-volume:/data \\

&nbsp; ubuntu

This mounts my-volume to the /data directory inside the container.



Step 3: Verify Volume Mount

Enter the container:

docker exec -it my-container bash

Inside container:

cd /data

echo "Hello from volume" > test.txt

Exit and re-run container:

Even if you stop and delete the container, the data in my-volume remains.



Step 4: Check Volume from Another Container

You can access the same volume from another container:



docker run --rm -v my-volume:/data alpine cat /data/test.txt

List and Inspect Volumes

List volumes:

docker volume ls

Inspect a volume:

docker volume inspect my-volume

To Delete a Volume (when not in use)

docker volume rm my-volume.



**8.Docker Compose for multi-container applications, Docker security best practices.**

1\. Docker Compose for Multi-Container Applications

Docker Compose is a tool to define and manage multi-container applications using a YAML file (docker-compose.yml).



Example: Node.js + MongoDB

File structure:



perl

my-app/

├── docker-compose.yml

├── backend/

│   ├── Dockerfile

│   └── app.js

docker-compose.yml:

yaml

version: '3.8'



services:

&nbsp; backend:

&nbsp;   build: ./backend

&nbsp;   ports:

&nbsp;     - "3000:3000"

&nbsp;   depends\_on:

&nbsp;     - mongo

&nbsp;   networks:

&nbsp;     - app-network



&nbsp; mongo:

&nbsp;   image: mongo:latest

&nbsp;   volumes:

&nbsp;     - mongo-data:/data/db

&nbsp;   networks:

&nbsp;     - app-network



volumes:

&nbsp; mongo-data:



networks:

&nbsp; app-network:

backend/Dockerfile:

dockerfile

FROM node:18-alpine

WORKDIR /app

COPY . .

RUN npm install

CMD \["node", "app.js"]

To run the app:

docker-compose up --build

Benefits of Docker Compose:

One command to manage all containers.



Defines volumes, networks, dependencies.



Easy to scale services using docker-compose up --scale.



2\. Docker Security Best Practices

Here are the key security measures you should follow when using Docker:



Image Security

Use official images or trusted sources.



Keep images minimal (e.g. alpine) to reduce attack surface.



Regularly scan images using tools like:

docker scan <image>

Container Isolation

Run containers as non-root user (use USER in Dockerfile).



Limit privileges with:



docker run --read-only --cap-drop all ...

Runtime Security

Use --read-only, --no-new-privileges, and resource limits:



docker run --memory 512m --cpus="1.0" ...

Network Security

Use custom bridge networks.



Avoid exposing containers directly unless necessary.



Secrets \& Configs

Never hard-code secrets in Dockerfiles.



Use environment variables or secret managers (e.g., Docker Secrets, Vault).



Host Security

Keep Docker updated.



Protect Docker socket (/var/run/docker.sock).



Monitor with tools like Docker Bench for Security.




































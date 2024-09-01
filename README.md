# Full-Stack FastAPI and React Application Deployment

![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)
![Maintenance](https://img.shields.io/badge/Maintained%3F-yes-green.svg)

Welcome to the Full-Stack FastAPI and React template repository. This repository serves as a demo application for showcasing how to set up and run a full-stack application with a FastAPI backend and a ReactJS frontend using ChakraUI, and dockerize said application to be deployed locally and to a cloud instance.

## Project Structure

The repository is organized into two main directories:

- **frontend**: Contains the ReactJS application.
- **backend**: Contains the FastAPI application and PostgreSQL database integration.

Each directory has its own README file with detailed instructions specific to that part of the application.

## Getting Started

To get started with this template, please follow the instructions in the respective directories:

- [Frontend README](./frontend/README.md)
- [Backend README](./backend/README.md)

## Dockerizing the Full Stack Application Locally

After completing the getting started step above, the following steps are taken to dockerize the application:


### Step 1: Build the Frontend Image in the `frontend` Directory

- Build the `Dockerfile` that contains all the **Node** dependencies and executables needed to install and run the **React** frontend.
- To do this, navigate to the frontend directory and build the Dockerfile.

```sh
cd frontend
```

- Test if the image can build successfully by running the command below:

```sh
docker build -t <image_name> .
```

### Step 2: Build the Backend Image in the `backend` Directory

- Build the `Dockerfile` that contains all the **Python Poetry** dependencies and executables needed to run the **FastAPI** backend.
- To do this, navigate to the backend directory and build the Dockerfile. Ensure that you are in the root directory before navigating.

```sh
cd backend
```

- Test if the image can build successfully by running the command below:

```sh
docker build -t <image_name> .
```

- Make the necessary changes in the `.env` files which will be referenced by `docker-compose` to build the application.

### Step 3: Build the `docker-compose.yaml` file
- To do this, ensure you are in the root directory before running this command.
  
```sh
docker-compose up -d
```

- The docker-compose.yml references the Dockerfiles in the `backend` and `frontend `directories to run as containers for the **backend** and **frontend** services respectively.

- **Traefik** runs as the Proxy Manager and is accessible via the subdomain `proxy.domain`.

- **Adminer** runs as the database manager, and is accessible via the subdomain `db.domain` which is properly connected to the **PostgresSQL** database.

_This is the configuration of the [docker-compose.yaml](./docker-compose.yaml) file_.


### Step 4: Verifying the Application

After a successful run, go to your web browser and access the following urls:

```
http://localhost
```
You should see a login page, where you can login using the superuser details in the .env file in the backend folder. If you can successfully login, then the frontend and backend container services are successfully communicating with each other.

```
http://localhost/api
```
This should show you the backend api service.

```sh
http://localhost/docs
```
This should show the docs for the api service.

```sh
http://localhost/redoc
```
This should show the redoc for the api service.

```sh
http://db.localhost
```
You should see a login page for adminer, you are to use the necessary database details provided in the .env file in the backend folder. If you can successfully login, and can see three already existing tables, then the postgresql container service is up and running successfully. 

```sh
http://proxy.localhost
```
You should see the traefik dashboard that manages the proxies for the domains.

_**Note**: The login details can be found in the `.env` file in the `backend` directory._

## Deploying & Hosting The Dockerized Application on AWS EC2

### Prerequisite
- a domain name (i.e. used for DNS configuration)

On AWS, `Route53` will be leveraged for DNS Routing. The steps taken to achieve this are stated below:

### Step 1: Install Docker on the EC2 Instance and clone the [repository](https://github.com/Data-Bishop/Dockerized-Full-Stack-Web-Application-Deployment.git)

- Spin up an EC2 Instance and run the commands to install `docker` and `docker-compose`.

```sh
cat <<EOF | tee docker.sh
#!/bin/bash
sudo apt-get update
sudo apt-get -y install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

sudo usermod -aG docker $USER

newgrp docker
EOF
```
_This is a bash script to install docker_.

```sh
bash docker.sh && sudo apt install docker-compose -y
git clone <dockerized-full-stack-application-url>
```

### Step 2: Update the `docker-compose.yaml` file with SSL & Subdomain Redirects

- Traefik offers a codebase approach directly in the `docker-compose.yaml` file and that's why it is the preferred Proxy Manager for implementing this project. 

- Configure Traefik to redirect `HTTP`and `www` requests to `HTTPS` and `non-www`.

- Ensure that the `frontend`, `backend`, and `adminer `routers are listening on the `websecure` entry point and using the TLS certresolver `myresolver`.

- _**Note**: The domain names must be consistent in the `docker-compose.yaml` file and `.env` files to avoid errors_.

### Step 3: Configure & Add DNS Records

- On the AWS console, navigate to `Route 53` and create a `Public Hosted` zone.

- Enter your domain name and click on create hosted zone.

- Copy the 4 values of **NS Records** in the hosted zone and paste them on the **DNS Nameserver Records** on the Web Hosting platform your domain was purchased.

- On the newly created hosted zone, create `A Record` for each subdomain routing to the **Public IPv4 Address** of the `EC2 Instance`:
    1. yourdomain.com
    2. www.yourdomain.com
    3. db.yourdomain.com
    4. proxy.yourdomain.com

_**Note**: After creation, a prompt will allow you to view the records' status to verify if they have been synchronized_.

- DNS changes can take some time to propagate. [WhatsMyDNS](https://www.whatsmydns.net/) validates if the DNS records have propagated globally, use it to test the 4 domains.

### Step 4: Deploy the Application

- SSH into the EC2 Instance and cd into the directory of the repo you clone.

- Run the following command to deploy the application:

```sh
docker-compose up -d
```

- The application should be running and can be assessed by anyone connected to the internet once they have your domain name.

Feel free to reach out if you encounter any issues.

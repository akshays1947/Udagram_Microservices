# Udagram Image Filtering Microservice

Udagram is a simple cloud application developed alongside the Udacity Cloud Engineering Nanodegree. It allows users to register and log into a web client, post photos to the feed, and process photos using an image filtering microservice.

The project is split into three parts:
1. [The Simple Frontend](/udacity-c3-frontend)
A basic Ionic client web application which consumes the RestAPI Backend. 
2. [The RestAPI Feed Backend](/udacity-c3-restapi-feed), a Node-Express feed microservice.
3. [The RestAPI User Backend](/udacity-c3-restapi-user), a Node-Express user microservice.

## Getting Setup

> _tip_: this frontend is designed to work with the RestAPI backends). It is recommended you stand up the backend first, test using Postman, and then the frontend should integrate.

### Installing Node and NPM
This project depends on Nodejs and Node Package Manager (NPM). Before continuing, you must download and install Node (NPM is included) from [https://nodejs.com/en/download](https://nodejs.org/en/download/).

### Installing Ionic Cli
The Ionic Command Line Interface is required to serve and build the frontend. Instructions for installing the CLI can be found in the [Ionic Framework Docs](https://ionicframework.com/docs/installation/cli).

### Installing project dependencies

This project uses NPM to manage software dependencies. NPM Relies on the package.json file located in the root of this repository. After cloning, open your terminal and run:
```bash
npm install
```
>_tip_: **npm i** is shorthand for **npm install**

### Setup Backend Node Environment
You'll need to create a new node server. Open a new terminal within the project directory and run:
1. Initialize a new project: `npm init`
2. Install express: `npm i express --save`
3. Install typescript dependencies: `npm i ts-node-dev tslint typescript  @types/bluebird @types/express @types/node --save-dev`
4. Look at the `package.json` file from the RestAPI repo and copy the `scripts` block into the auto-generated `package.json` in this project. This will allow you to use shorthand commands like `npm run dev`


### Configure The Backend Endpoint
Ionic uses enviornment files located in `./src/enviornments/enviornment.*.ts` to load configuration variables at runtime. By default `environment.ts` is used for development and `enviornment.prod.ts` is used for produciton. The `apiHost` variable should be set to your server url either locally or in the cloud.

***
### Running the Development Server
Ionic CLI provides an easy to use development server to run and autoreload the frontend. This allows you to make quick changes and see them in real time in your browser. To run the development server, open terminal and run:

```bash
ionic serve
```

### Building the Static Frontend Files
Ionic CLI can build the frontend into static HTML/CSS/JavaScript files. These files can be uploaded to a host to be consumed by users on the web. Build artifacts are located in `./www`. To build from source, open terminal and run:
```bash
ionic build
```


### Setup Docker environment
You will need to install Docker Desktop application (for windows). Instructions for configuring it can be found in the [Docker Desktop on Windows](https://docs.docker.com/docker-for-windows/install/).
Once docker desktop is up and running, go to the settings and create a docker hub acccount. This account will be used as a repository for storing docker images.

Open new terminal into the project directory and run:
1. Create a Dokerfile: `touch Dockerfile`
2. Build image for a service (say user service): `docker build -t dockerRepo/udacity-c3-restapi-user .`
3. Get a list of the created images: `docker images`
4. Remove an image: `docker image rm -f <image_name>` and then run `docker image prune`
5. To create and run an image: `docker run --rm --publish 8080:8080 -v $HOME\.aws:/root/.aws -e POSTGRESS_HOST -e POSTGRESS_USERNAME  -e POSTGRESS_PASSWORD -e POSTGRESS_DB -e AWS_REGION -e AWS_PROFILE -e AWS_BUCKET -e JWT_SECRET --name feed dockerRepo/udacity-restapi-feed`
6. To verify if the container is working fine:  `curl http://localhost:8080/api/v0/feed`
7. Build the images for each of our defined services, using the command: `docker-compose -f docker-compose-build.yaml build --parallel`
8. To start the system, run a container for each of our defined services, in the attached mode: `docker-compose up`
9. Alternatively, you may use detached mode to run containers in the background: ` docker-compose up -d`
10. To stop the container: `docker-compose stop`
11. To remove (and stop) the container `docker-compose down`
***

### Setup Kubernetes environment:

You will need to install the Kubectl command line. Instructions of which can be found in [Kubernetes documentation](https://kubernetes.io/docs/tasks/tools/install-kubectl/).

Open new terminal into the project directory and run:
1. Generate encrypted values for aws credentials, Database User Name, and Database Password and put the values into aws-secret.yaml and env-secret.yaml files.
2. Load secret files:
- `kubectl apply -f aws-secret.yaml`
- `kubectl apply -f env-secret.yaml`
3. Load config map: 
- `kubectl apply -f env-configmap.yaml`
4. Apply deployments:
- `kubectl apply -f backend-feed-deployment.yaml`
- `kubectl apply -f frontend-deployment.yaml`
- `kubectl apply -f backend-user-deployment.yaml`
5. Apply services:
- `kubectl apply -f backend-feed-service.yaml`
- `kubectl apply -f backend-user-service.yaml`
- `kubectl apply -f frontend-service.yaml`
6. Deploy reverseproxy, has to be done after other services are running:
- `kubectl apply -f reverseproxy-deployment.yaml`
- `kubectl apply -f reverseproxy-service.yaml`
7. Perform port forwarding (each needs to be run in a separate terminal window and left running):
- `kubectl port-forward service/frontend 8100:8100`
- `kubectl port-forward service/reverseproxy 8080:8080`

### Check kubernetes cluster status

1. `kubectl get nodes`
2. `kubectl get pod --all-namespaces`
3. `kubectl get svc`



### Resources for setting up and using KOPS.

Refer to the following articles for getting information about the processes and dependencies involved while setting up KOPS.

1. [Setup Kubernetes On AWS Using KOPS](https://medium.com/cloud-academy-inc/setup-kubernetes-on-aws-using-kops-877f02d12fc1).
2. [Install Kubernetes on AWS using KOPS](https://www.studytrails.com/devops/kubernetes/install-kubernetes-on-aws-using-kops/).

### Continuous Integration and Continuous Deployment.
1. Travis CI is setup to monitor for updates to any branches and will automatically build and deploy the Docker containers.
2. For this to work setup the DOCKER_PASSWORD and DOCKER_USERNAME in the environment variables of Travis CI. You can find the instructions for setting it up at [Using Docker in Builds](https://docs.travis-ci.com/user/docker/).

### Deployment of new application version without downtime.

To deploy a new version of the application without downtime. Following changes are to be done to the code:
1. Add "strategy" under spec to the application deployment files:
```bash
    strategy:
        type: RollingUpdate
        rollingUpdate:
            maxSurge: 1
            maxUnavailable: 0
```

2. To ensure zero downtime add the following under spec.container.image in deployment files:
```bash
    readinessProbe:
        httpGet:
            path: /
            port: 8080
        initialDelaySeconds: 5
        periodSeconds: 5
        successThreshold: 1
```
3. Add a new version under metadata.labels in reverseproxy deployment file:
```bash
version: 2
```

>> For these changes to take place, you must reapply the affected deployments.

### A/B deployment of the application

The folder "udacity-c3-restapi-ab" contains the same code as of "udacity-c3-restapi" but is named differently to show it is a different version of "udacity-c3-restapi". This has been done to have two docker containers with different codes to show two versions - 'A' and 'B' of the same application can run simultaneously and serve the traffic.


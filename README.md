
# Jenkins Setup with Docker-Compose

This is repository for auotamet to jenkins setup on container with docker-compose.

## Prerequisites

- docker
- docker-compose


## Usage

1. git clone https://github.com/gustysap/jenkins-setup-docker.git
2. cd jenkins-setup-docker
3. docker-compose up -d

## Description

Need to create ```casc_config.yaml``` for the configuration code to make it easier for us to automate the Jenkins setup and create ```plugins.txt``` for a list of plugins that we can install at the beginning of the jenkins image running

### casc_config.yaml

We can setup user and password to login on this side which will get the value from a key environment variable. And we also have to setup the authorization matrix from the user side. For the code below I setup this user as administrator :

```
jenkins:
  securityRealm:
    local:
      allowsSignup: false
      users:
       - id: ${JENKINS_ADMIN_USER}
         password: ${JENKINS_ADMIN_PASSWORD}
  authorizationStrategy:
    globalMatrix:
      permissions:
        - "Overall/Administer:admin"
        - "Overall/Read:authenticated"
```

### plugins.txt

This is a list of plugins that we will install in a custom Jenkins image.
You can add it yourself if needed. Here what must be installed is
- configuration-as-code plugin for configuration as code
- matrix-auth plugin for matrix on configuration as code which we have setup in casc_config.yaml
- credentials-binding plugin
- pam-auth plugin

the rest are extras like git plugin and kubernetes plugin as needed

```
configuration-as-code:latest
git:latest
pipeline-stage-view:latest
kubernetes:latest
credentials-binding:latest
matrix-auth:latest
pam-auth:latest

```
### Dockerfile

Create ```Dockerfile``` for create custom jenkins image
```
FROM jenkins/jenkins # base image jenkins latest
ENV JAVA_OPTS -Djenkins.install.runSetupWizard=false  # this environment vairable for skip setup wizard on first run jenkins image
ENV CASC_JENKINS_CONFIG /var/jenkins_home/casc_config.yaml # this environment variable for set path casc ( configuration as code ) yaml. 
COPY plugins.txt /usr/share/jenkins/ref/plugins.txt # copy plugins.txt to path on docker image
RUN /usr/local/bin/install-plugins.sh < /usr/share/jenkins/ref/plugins.txt # run bash script for install plugins from the list file
COPY casc_config.yaml /var/jenkins_home/casc_config.yaml # copy casc_config.yaml to environment path CASC_JENKINS_CONFIG
```

### docker-compose.yml

this is the docker compose yaml configuration. In the configuration there are 2 environment variables that you can setup as needed for credentials or username and password.

- JENKINS_ADMIN_USER=( ENTER YOUR VALUE ADMIN USER )
- JENKINS_ADMIN_PASSWORD ( ENTER YOUR VALUE ADMIN PASSWORD )



```
#version of docker-compose
version: '3.7'
# we call an application container in docker compose is service
services:
# jenkins is the name of service
  jenkins:
# setup environment variable for casc_config.yaml need to setup username and password administrator
    environment:
      - JENKINS_ADMIN_USER=admin 
      - JENKINS_ADMIN_PASSWORD=password
# for build the image, this is the path where the dockerfile is located
    build: ./
# this the name of jenkins image custom
    image: jenkins:custom
# this is to give container privileges and which user the container runs on
    privileged: true
    user: root
# this is for setup listening port like port forward (listen port host):(listen port container)
    ports:
      - 8081:8080
      - 50000:50000
# this is for give the container name
    container_name: jenkins
# this is for setup volume, because docker or containers are immutable, we must bind the volume to the path on the host so that when the container is deleted, the data remains persistent. 
# /pathhost:/pathcontainer
    volumes:
      - ~/jenkins:/var/jenkins_home
      - ./casc_config.yaml:/var/jenkins_home/casc_config.yaml
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/local/bin/docker:/usr/local/bin/docker
```

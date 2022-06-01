# Jenkins Cookbook

I am in the process of reconfiguring CI/CD for a new project I am working on, and decided to document for ease of future-reuse, all the usual stuff I take notes on, but now for [Jenkins](https://www.jenkins.io).

<!--BEGIN TOC-->
## Table of Contents
1. [Jenkins quick-start with Docker](#jenkins-quick-start-with-docker)
    1. [Creating the prerequisites](#creating-the-prerequisites)
    2. [Running Jenkins](#running-jenkins)
2. [Python-Jenkins worked example](#python-jenkins-worked-example)
3. [Jenkins with Gitea](#jenkins-with-gitea)

<!--END TOC-->

## Jenkins quick-start with Docker
The latest image for jenkins is [`jenkinsci/blueocean`](https://hub.docker.com/r/jenkinsci/blueocean/), obtainable with the usual
```bash
docker pull jenkinsci/blueocean
```
We will also use the `docker:dind` image:
```bash
docker image pull docker:dind
```

This setup follows the [official documentation](https://www.jenkins.io/doc/book/installing/).

### Creating the prerequisites
Jenkins requires a docker network and (recommended) two docker volumes. We can create the bridged network and simple default volumes with
```bash
docker network create jenkins
# for TLS certificates
docker volume create jenkins-docker-certs
docker volume create jenkins-data
```
We will require use of Docker-In-Docker `docker:dind`, so that Jenkins can spin up and manage containers. We run this command to configure this:

```bash
docker container run \
  --name jenkins-docker \
  --rm \
  -d \
  --privileged \
  --network jenkins \
  --network-alias docker \
  -e DOCKER_TLS_CERTDIR=/certs \
  -v jenkins-docker-certs:/certs/client \
  -v jenkins-data:/var/jenkins_home \
  -p 2376:2376 \
  docker:dind
```

### Running Jenkins
To start the jenkins service, we will use
```bash
docker container run \
  --name jenkins-blueocean \
  --rm \
  -d \
  --network jenkins \
  -e DOCKER_HOST=tcp://docker:2376 \
  -e DOCKER_CERT_PATH=/certs/client \
  -e DOCKER_TLS_VERIFY=1 \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins-data:/var/jenkins_home \
  -v jenkins-docker-certs:/certs/client:ro \
  jenkinsci/blueocean
```

Then visit [`http://localhost:8080`](http://localhost:8080) to enter setup.


## Python-Jenkins worked example
Following from tutorials on the Jenkins website, primarily [this example](https://www.jenkins.io/doc/tutorials/build-a-python-app-with-pyinstaller/).

## Jenkins with Gitea
If you're hosting a [Gitea](https://gitea.io/en-us/) server, you may wish to fully automate local CI/CD with Jenkins. In order to do that, we log into the Jenkins server with an admin account, and under the *Manage Jenkins* tab, navigate to *Manage Plugins*, and from there, install the Gitea plugin.

Under *Manage Jenkins* again, this time *Configure System*, scroll down to Gitea and add a Gitea server. If you want to enable webhook automation, be sure to tick the box, and create a Jenkins account on Gitea for Jenkins to use. NB: this account must be an administrator. Apply and save your changes.

Next, create a new (mutlibranch) Pipeline, and under *Add Source*, select Gitea and your configured server, along with the respective credentials if you enabled webhook management.

You don't directly copy a link, but rather enter the username of the account holding the repository you wish to automate, and then select the repository from the dropdown menu.

And that's it -- Jenkins should start running your Jenkinsfile.

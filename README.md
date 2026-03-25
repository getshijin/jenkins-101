# Jenkins 101

This repository contains a small Jenkins demo project for learning how to run a simple Python application through a Jenkins pipeline running in Docker.

## What's included

- `Jenkinsfile` – declarative pipeline with build, test, and deliver stages.
- `Jenkinsfile.template` – pipeline template/reference file.
- `Dockerfile` – custom Jenkins Blue Ocean image with Docker CLI installed.
- `helloworld.py` – minimal Python smoke-test script.
- `myapp/hello.py` – simple CLI app built with `fire`.
- `myapp/requirements.txt` – Python dependency list for the sample app.
- `readme.md` – original course notes and setup commands kept for reference.

## Sample application

The demo application lives in `myapp/` and exposes a tiny command-line interface:

```bash
cd myapp
pip install -r requirements.txt
python3 hello.py
python3 hello.py --name=Brad
```

Expected output:

- `Hello World!`
- `Hello Brad!`

There is also a simple `helloworld.py` script at the repository root that prints `Hello world`.

## Jenkins pipeline overview

The `Jenkinsfile` is configured to run on an agent labeled `docker-agent-python` and includes three stages:

1. **Build** – installs Python dependencies from `myapp/requirements.txt`.
2. **Test** – runs the sample CLI with default and named arguments.
3. **Deliver** – placeholder stage for deployment or packaging work.

The pipeline also includes SCM polling (`* * * * *`) for frequent change detection in demo environments.

## Running Jenkins with Docker

This repository is set up to work with a custom Jenkins image based on `jenkins/jenkins:2.332.3-jdk11` plus Blue Ocean and Docker CLI support.

### Build the image

```bash
docker build -t myjenkins-blueocean:2.332.3-1 .
```

### Create the Docker network

```bash
docker network create jenkins
```

### Start Jenkins

```bash
docker run --name jenkins-blueocean --restart=on-failure --detach \
  --network jenkins --env DOCKER_HOST=tcp://docker:2376 \
  --env DOCKER_CERT_PATH=/certs/client --env DOCKER_TLS_VERIFY=1 \
  --publish 8080:8080 --publish 50000:50000 \
  --volume jenkins-data:/var/jenkins_home \
  --volume jenkins-docker-certs:/certs/client:ro \
  myjenkins-blueocean:2.332.3-1
```

After the container starts, retrieve the initial admin password with:

```bash
docker exec jenkins-blueocean cat /var/jenkins_home/secrets/initialAdminPassword
```

Then open Jenkins at:

```text
http://localhost:8080/
```

## Typical Jenkins job flow

1. Build and start the Jenkins container.
2. Configure or connect an agent with the label `docker-agent-python`.
3. Create a Pipeline job that points at this repository.
4. Let Jenkins discover `Jenkinsfile` and run the stages.
5. Review the console output for build and test results.

## Notes

- This repository appears to accompany a Jenkins tutorial/course.
- The original setup notes, registry references, and extra Docker helper commands remain in `readme.md`.
- You can adapt the deliver stage to publish artifacts, build Docker images, or deploy an application.

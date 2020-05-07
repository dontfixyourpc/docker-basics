
# How to Docker:  Create a basic Dockerfile

This is a short *simplified* example which demonstrates how to build a Dockerimage via a Dockerfile

See more information here https://github.com/grafana/loki/tree/master/docs/clients/promtail

### 1. Vanilla Dockerfile to build promtail

A Vanilla Dockerfile for promtail could look like this

```dockerfile
FROM ubuntu:18.04

RUN apt-get update && \
  apt-get install -qy \
  tzdata ca-certificates libsystemd-dev && \
  rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*
ENV PROM_VERSION=/v1.4.1/promtail-linux-amd64.zip
ENV PROM_URL=https://github.com/grafana/loki/releases/download/${PROM_VERSION}
RUN apt-get update && apt-get -y install curl unzip && curl -L ${PROM_URL} -o promtail-linux-amd64.zip && unzip promtail-linux-amd64.zip \
  && chmod 750 promtail-linux-amd64 && mv /promtail-linux-amd64 /usr/bin/promtail
EXPOSE 9080
COPY promtail-docker-config.yaml /etc/promtail/docker-config.yaml
CMD ["-config.file=/etc/promtail/docker-config.yaml"]
ENTRYPOINT ["/usr/bin/promtail"]
```

#### Step-by-Step Description

1. As you can see in the first line there is a FROM, it is always the first line in a Dockerfile and 
tells docker which Base-Image to use for our build. There are official docker repositories at the Dockerhub
which can be downloaded and used. We will use the official Ubuntu image from there.
2. The first RUN command updates the packages sources from Ubuntu and installs needed packages for operations and to get and unpack promtail
3. EXPOSE sets up the port in localhost
4. The ENV's can be used to change the version and shows the url to download promtail from.
5. The second RUN command downloads promtail, unzips it and installs in
6. COPY puts the files into the correct folder
7. CMD in this particular case, shows promtails options
8. ENTRYPOINT is promtail itself (which then will use the CMD as options)


### 2. Docker config for the Image

There is a .yaml-file in the project that configures promtail. 
#### promtail-docker-config.yaml

```yaml
server: 
    http_listen_port: 9080
    grpc_listen_port: 0

positions:
    filename: /tmp/positions.yaml
    
clients: 
  - url: http://loki:3100/loki/api/v1/push
    
scrape_configs:
  - job_name: system 
    static_configs: 
      - targets:
            - localhost 
        labels:
            job: varlogs
            __path__: /var/log/*log

```
#### Step-by-Step Description:

This registry provides key settings for the Docker Container.

1. Sets up http-listen-port to 9080 and grpc-port to 0
2. Sets up of the positions the file
3. Sets up of clients the url 
4. Setting up scrape configurations -> static configurations, define localhost as target etc.

# How To Docker: Using a Docker Registry 

## Content:

- Reasons to use a Docker Registry
- Create a local Docker Registry

### Reasons to use a Docker Registry:

1. Control where *"homemade"* images are being stored
2. Good versioning possibilities
3. Avoiding copyright issues, by storing local own images in the registry
4. For managing purposes: Using like a lib for containers

### Deploy a local Docker Registry

There is no need to write a Dockerfile for a registry. 
Just enter this command down bellow in the Terminal. (Make sure that docker is already installed)
#### Step 1: Run the registry

```bash 
$ docker run -d -p 3000:3000 --restart=always --name "myregistry" registry:2
```
##### Short description:
- With **-d** the container will run in background and print the container id
- **-p** sets up the port on localhost:3000
- defines the restart policy to always 
- sets up a name.(Disclaimer: Don`t use "myregistry" as name tag. It is just a placeholder )

#### Step 2: Push a image to the Docker Registry

Let´s assume that you already created images and you want to store them in your local registry.

### Step 1: Show all your images
```bash
$ docker images 
```
### Step 2: Pick one image and tag them
```bash
$ docker tag "yourimage : version" localhost:3000/"yourimage"
```
### Step 3: Push it into your registry
```bash
$ docker push localhost:3000/"yourimage"
```
### What you can do is...
```bash
# remove the locally cached images 

$ docker image remove "yourimage"

$ docker image remove localhost:3000/"yourimage"

```
Now you can *pull* your image from your local registry.
``

### How to stop the registry? 

```
$ docker stop "myregistry"
```
**Disclaimer: Everything in "" is just a placeholder for educational purposes!**

---


### Disclaimer: 

As said before this is just a simplified version for educational purposes. For Docker Best Practices, please follow this link : <https://docs.docker.com/develop/develop-images/dockerfile_best-practices/>


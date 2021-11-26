# Robot Shop for Power Linux (ppc64le architecture)

This is a fork of original robot-shop repo from Instana team. A customer wanted me to demonstrate Instana capabilities monitoring applications running on Power Systems, so naturally I wanted to setup Robot Shop on some kind of Linux running on Power servers to be monitored by Instana agent.

Unfortunately the official docker images are compatible only with classical amd64 architecture and I was unable to find any resources, that would guide me to set it up on Power System environment.

I took the time to go over each Dockerfile and fix the source images, so that it is possible to deploy Robot Shop on ppc64le architecture.

## Disclaimer

This is in no way a official guide on how to setup Robot Shop on Power. This is more of a description of what I did with my environment to get the Robot Shop up & running.

## My environment description

- Server: Power S822l
- Architecture: ppc64le (power pc 64 little endian)
- Power VM: RHEL 8.3 with minimal install (prepared template, cleanly installed ISO from RedHat software download portal)
- Hardware: 2 vCPU, 8 GB RAM, 50 GB HDD

## Install

### Prerequsites
There are some prerequisites you need to install for Robot Shop to work. First and most important is obviously Docker. You can follow any tutorial and it should work for ppc64le. I did not encouter any trouble here. I have followed this documentation [https://docs.docker.com/engine/install/rhel/](https://docs.docker.com/engine/install/rhel/)

The second is Docker-Compose. You need to use the Python (pip) install method, otherwise the installation will fail. I have followed official documentation, using the `Alternative install options` method. Please note, that you probably have to install Python and Pip first, to make this work. Documentation can be found here: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)

```
pip3 install docker-compose
```

At the end you need to download Git to clone this repo.

```
yum install git
git clone <this repo url>
cd <this repo folder>
```

### Robot Shop deployment
If you did not done so yet, navigate to the folder of cloned repo and simply run following commands. First you will need to build the images locally.

```
docker-compose build
```

Fire up download images with `-d` flag to run on background.

```
docker-compose -f docker-compose.yaml -f docker-compose-load.yaml up -d
```

Run following command to check if everything went smooth. You can try to debug containers that `exited`.

```
docker ps -a
docker logs <container ID>     <---    use for container troubleshooting
```

## Disabled tracing, cause NOT supported?
I was not able to get Nginx tracing running on ppc64le architecture. I had to heavily modify the Dockerfile of `web` service component to disable tracing. I am not sure if this is not supported by Instana agent currently, but during deployment of `rs-web` service I ran into issues. The cause was probably non-compatible `instanalib.so` file, that is copied during the deployment into Nginx library folder. Unfortunately this file is available only for amd64 architecture, so no tracing yet. 

The exact error I was getting:

```
[emerg] 1#1: dlopen() "/etc/nginx/modules/ngx_http_opentracing_module.so" failed (/etc/nginx/modules/ngx_http_opentracing_module.so: cannot open shared object file: No such file or directory) in /etc/nginx/nginx.conf:2
```
I have manually went inside the container and verified that the file existed. This file is downloaded from Instana online repos and copied over by the Dockerfile but for some reason is not recognized by the Nginx server. I have fiddled around with different versions of the Nginx server and Alpine base image but simply could not get it to work. Otherwise everything else works fine and for short demo it is more than enough.

## Dockerfile changes overview

In this section I want to highlight the changes that I made to individual Dockerfiles. Mostly it is just the change of source image, to be compatible with ppc64le architecture.

| Name      | Previous Image | Current Image              | Notes |
|-----------|----------------|----------------------------|-------|
| Cart      | node:14        | node:14-buster             |   |
| Catalogue | node:14        | node:14-buster             |   |
| Dispatch  | golang:1.17    | golang:1.17-buster         |   |
| Mongo     | mongo:5        | ibmcom/mongodb-ppc64le:4.4 |   |
| MySql     | mysql:5.7      | mariadb                    | Only old versions of mysql exists for ppc64le. Had to switch to mariadb |
| Payment   | python:3.9     | python:3.9                 | No changes  |
| Ratings   | php:7.4-apache | php:7.4-apache             | No changes  |
| Shipping  | openjdk:8-jdk  | ppc64le/openjdk:8-jdk      |   |
| User      | node:14        | node:14-buster             |   |
| Web       | alpine         | nginx:1.20.1               | After removing the tracing part, alpine is not needed anymore |


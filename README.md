# Robot Shop for Power Linux (ppc64le architecture)

This is a fork of original robot-shop repo from Instana team. A customer wanted me to demonstrate Instana capabilities monitoring applications running on Power Systems, so naturally I wanted to setup Robot Shop on some kind of Linux running on Power servers to be monitored by Instana agent.

Unfortunately the official docker images are compatible only with classical amd64 architecture and I was unable to find any resources, that would guide me to set it up on Power System environment.

I took the time to go over each Dockerfile and fix the source images, so that it is possible to deploy Robot Shop on ppc64le architecture.

## Disclaimer

This is in no way a official guide on how to setup Robot Shop on Power. This is more of a description of what I did with my environment to get the Robot Shop up & running. Please note, that I had to disable tracing for Robot Shop webpage, because Instana currently (24.11.2021) doesn't support tracing on ppc64le architecture. When it was enable I had issues with rs-web image deploying, because of nginx failing to start. The cause was probably non-compatible `instanalib.so` file, that is copied during the deployment into nginx library folder. Unfortunately this file is available only for amd64 architecture, so no tracing yet.

## My environment description

- Server: Power S822l
- Architecture: ppc64le (power pc 64 little endian)
- Power VM: RHEL 8.3 with minimal install (prepared template, cleanly installed ISO from RedHat software download portal)
- Hardware: 2 vCPU, 8 GB RAM, 50 GB HDD

## Install


### Prepare environment


### Deploy Robot Shop


### Deploy Instana Agent


## Disable tracing, cause NOT supported


## Dockerfile changes overview

In this section I want to highlight the changes that I made to individual Dockerfiles. Mostly it is just the change of source image, to be compatible with ppc64le architecture.

| Name      | Previous Image | Now Image                  | Notes |
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


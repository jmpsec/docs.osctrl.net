---
title: "Using Docker"
description: "Run osctrl and all its components locally with Docker Compose using the docker-compose-dev.yml stack."
sidebar:
  order: 2
---

You can use docker to run **osctrl** and all the components are defined in the `docker-compose-dev.yml` that ties all the components together, to serve a functional deployment.

Ultimately you can just execute `make docker_dev` and it will automagically build and run **osctrl** locally in docker, for development purposes.

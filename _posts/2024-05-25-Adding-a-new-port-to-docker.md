---
title: Adding a New Port to an Existing Docker Container
date: 2024-06-01 11:31:00 -0700
categories: [docker]
tags: 
Pin:
---

## Overview

This guide explains how to add a new port to an existing Docker container using `alpine/socat` and Docker networking.

## Steps

1. **Create a Docker Network**

    First, create a Docker network to connect your containers.

    ```bash
    docker network create my-network
    ```

2. **Connect the Existing Container to the Network**

    Connect your existing container to the newly created network.

    ```bash
    docker network connect my-network my-container
    ```

3. **Run `alpine/socat` to Forward the Port**

    Run a new Docker container using the `alpine/socat` image to forward the desired port to the existing container. This example forwards port `1883`.

    ```bash
    docker run -d --name rabbitmq-mqtt-proxy --network my-network -p 1883:1883 alpine/socat tcp-listen:1883,fork,reuseaddr tcp-connect:my-container:1883
    ```

    - `-d`: Run the container in detached mode.
    - `--name rabbitmq-mqtt-proxy`: Name the new container `rabbitmq-mqtt-proxy`.
    - `--network my-network`: Connect the new container to `my-network`.
    - `-p 1883:1883`: Map port `1883` on the host to port `1883` in the container.
    - `alpine/socat`: Use the `alpine/socat` image.
    - `tcp-listen:1883,fork,reuseaddr`: Listen on port `1883` and forward connections.
    - `tcp-connect:my-container:1883`: Connect to port `1883` on the existing container named `my-container`.

## Summary

By following these steps, you can forward a new port to an existing Docker container using `alpine/socat` and Docker networking. This allows you to expose additional services without modifying the original container configuration.


---
layout: post
title:  "Building ARM images on DockerHub"
date:   2018-12-09 22:20:00 +0100
categories: docker dockerhub github
---

While I am about to install more and more containers on my Raspberry PI driven Kubernetes cluster at home, I wanted to have DockerHub create those images. What I will describe here is nothing fancy and new - it's just the combination of what I found.

To run arm binaries on a x86 system we obviously need QEMU - and very conviniently there is already a docker container that can help us [multiarch/qemu-user-static](https://github.com/multiarch/qemu-user-static).

All we need to do is to register qemu like that:

```
docker run --rm --privileged multiarch/qemu-user-static:register
```

And then add the qemu-static binary to the docker container that we want to build.

The challenge hereby is that I up until this point didn't know that you can execute `pre_build` and `post_checkout` steps within the DockerHub CI.

All you have to do is to create a `hooks` directory and add the 2 files shown below to it
```
hooks
|-- post_checkout
`-- pre_build
```


post_checkout:
```
#!/bin/bash
# download 
curl -L https://github.com/multiarch/qemu-user-static/releases/download/v3.0.0/qemu-arm-static -o qemu-arm-static
chmod 755 qemu-arm-static
```

pre_build
```
#!/bin/bash
docker run --rm --privileged multiarch/qemu-user-static:register --reset
```

DockerHub will not download the qemu-arm-static binary and register it before each build.
What's left is to add the downloaded qemu-arm-static binary to the container (within the Dockerfile)

```
ADD qemu-arm-static /usr/bin
```


My GitHub repos for 2 of the images:
* [dasher-mqtt-rpi](https://github.com/seb-daehne/dasher-mqtt-rpi)
* [telegraf-rpi](https://github.com/seb-daehne/telegraf-rpi)

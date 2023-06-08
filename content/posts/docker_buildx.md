---
title: "Buildx: building multi-arch images"
date: 2021-01-29T12:00:00+01:00
publishdate: 2021-01-29
lastmod: 2023-06-08T10:11:12+01:00
draft: false
tags: ["docker", "arm64", "aarch64"]
categories: ["containers"]
author: "Carmelo C."
---

In this post I'll explore how to build images that can run on multiple architectures. My scenario is composed of the two following hosts:
- x86_64
  ```
  $ docker info | grep Architecture
   Architecture: x86_64

  $ cat /proc/cpuinfo | grep "model name" | uniq
  model name	: Intel(R) Core(TM) i5-3317U CPU @ 1.70GHz
  ```
- ARM
  ```
  $ docker info | grep Architecture
   Architecture: armv7l

  $ cat /proc/cpuinfo | grep "model name" | uniq
  model name	: ARMv7 Processor rev 4 (v7l)
  ```
Docker version being run is `24.0.2`.

### On x86_64
`hello.go`:
```
package main

import "fmt"

func main() {
    fmt.Println("Hello, world!\n")
}
```

`Dockerfile`:
```
FROM golang:alpine3.18 AS builder
COPY hello.go src/
RUN go build -o hello src/hello.go

FROM alpine:3.18
RUN mkdir /app
WORKDIR /app
COPY --from=builder /go/hello ./
CMD ["./hello"]
```

```
user@x86_64 $ docker build -t carmelo0x99/hellogo:1.0 -f Dockerfile.hello .

user@x86_64 $ docker push carmelo0x99/hellogo:1.0

user@x86_64 $ docker run --rm carmelo0x99/hellogo:1.0
Hello, world!
```

### Running the same image on ARM leads to...
```
user@arm $ docker run -it --rm carmelo0x99/hellogo:x86_64
WARNING: The requested image's platform (linux/amd64) does not match the detected host platform (linux/arm/v7) and no specific platform was requested
exec ./hello: exec format error
```
Unsurprisingly, because:
```
user@arm $ docker image inspect --format "{{.Architecture}}" carmelo0x99/hellogo:1.0
amd64
```

### Enter [Buildx](https://docs.docker.com/build/)
```
user@x86_64 $ docker buildx version
github.com/docker/buildx v0.10.5 86bdced

user@x86_64 $ docker buildx create --driver docker-container --bootstrap --use --name mybuilder
[+] Building 1.8s (1/1) FINISHED
 => [internal] booting buildkit                                                 1.8s
 => => pulling image moby/buildkit:buildx-stable-1\                             1.1s
 => => creating container buildx_buildkit_mybuilder0                            0.6s
mybuilder

user@x86_64 $ docker buildx ls
NAME/NODE    DRIVER/ENDPOINT             STATUS  BUILDKIT         PLATFORMS
mybuilder *  docker-container
  mybuilder0 unix:///var/run/docker.sock running v0.11.6          linux/amd64, linux/amd64/v2, linux/386
default      docker
  default    default                     running v0.11.7-0.2...   linux/amd64, linux/amd64/v2, linux/386
```

### NOTE: running with `--load` generates an error
```
user@x86_64 $ docker buildx build -t carmelo0x99/hellogo:2.0 --platform linux/amd64,linux/386,linux/arm/v7,linux/arm/v6 --load -f Dockerfile.hello .
[+] Building 0.0s (0/0)
ERROR: docker exporter does not currently support exporting manifest lists
```
[Buildx issue #59](https://github.com/docker/buildx/issues/59)

### `--push` works instead
```
user@x86_64 $ docker buildx build -t carmelo0x99/hellogo:2.0 --platform linux/amd64,linux/386,linux/arm/v7,linux/arm/v6 --push -f Dockerfile.hello .
[+] Building 289.6s (51/51) FINISHED
...
```
**NOTE**: this step takes a bit of time. Docker must run QEMU to build the images for the _other_ architectures.</br>

Problem is we need to pull the image back from Docker Hub.
```
user@x86_64 $ docker pull carmelo0x99/hellogo:2.0
2.0: Pulling from carmelo0x99/hellogo
8a49fdb3b6a5: Already exists
280c8b627803: Pull complete
4f4fb700ef54: Pull complete
03bc6fb78b91: Pull complete
Digest: sha256:b642d8947aa2ecce374b15a4ac767b7cb4d5f2fe81709702afb1091875e23696
Status: Downloaded newer image for carmelo0x99/hellogo:2.0
docker.io/carmelo0x99/hellogo:2.0
```

It's interesting to notice how the image _embeds_ several different flavours
```
user@x86_64 $ docker buildx imagetools inspect carmelo0x99/hellogo:2.0
Name:      docker.io/carmelo0x99/hellogo:2.0
MediaType: application/vnd.oci.image.index.v1+json
Digest:    sha256:b642d8947aa2ecce374b15a4ac767b7cb4d5f2fe81709702afb1091875e23696

Manifests:
  Name:        docker.io/carmelo0x99/hellogo:2.0@sha256:09198d4c23055f38c447a7e4b511777c42de5761d56007d52ca407492a17365d
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/amd64

  Name:        docker.io/carmelo0x99/hellogo:2.0@sha256:e4a13ef6f4c7d9b2336e5d4de57005b41705770475eae519632021741bba6c3c
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/386

  Name:        docker.io/carmelo0x99/hellogo:2.0@sha256:56a3397cb049baaaf2e8523fe7c1b3c4df2c900aab24a135bbc4c8f1bad6792e
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/arm/v7

  Name:        docker.io/carmelo0x99/hellogo:2.0@sha256:3ae0b2142f8a43209cdd1d577a309141e4b854c70839d8612621b16ae4ab5e89
  MediaType:   application/vnd.oci.image.manifest.v1+json
  Platform:    linux/arm/v6
...
```

Of course:
```
user@x86_64 $ docker run --rm carmelo0x99/hellogo:2.0
Hello, world!
```

### on ARM after buildx
We're good to go now. Same image we'd run on AMD/Intel/x86_64 runs **seamlessly** on ARM.
```
user@arm $ docker pull carmelo0x99/hellogo:2.0
2.0: Pulling from carmelo0x99/hellogo
e14425cf8fb9: Already exists
4d43510b4c85: Pull complete
4f4fb700ef54: Pull complete
1117a7594d95: Pull complete
Digest: sha256:b642d8947aa2ecce374b15a4ac767b7cb4d5f2fe81709702afb1091875e23696
Status: Downloaded newer image for carmelo0x99/hellogo:2.0
docker.io/carmelo0x99/hellogo:2.0

user@arm $ docker run carmelo0x99/hellogo:2.0
Hello, world!
```

Finally, notice how the images lookl different on the two architectures:
```
user@x86_64 $ docker image ls carmelo0x99/hellogo:2.0
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
carmelo0x99/hellogo   2.0       390a9611350b   6 minutes ago   9.17MB
```
... and...
```
user@arm $ docker image ls carmelo0x99/hellogo:2.0
REPOSITORY            TAG       IMAGE ID       CREATED         SIZE
carmelo0x99/hellogo   2.0       2d8c851a5cfb   4 minutes ago   6.59MB
```

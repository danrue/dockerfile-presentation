This repo is based on a 5 minute presentation which itself was inspired by Abby
Fuller's Dockercon 2017 "Creating Effective Images" talk.

Run the following three commands while making the changes between the
Dockerfiles:

```sh
    docker build -t pyjokes .
    docker run pyjokes
    docker images pyjokes
```

## Dockerfile.1

Initial Dockerfile. Runs "pyjoke" and pipes the output to cowsay -f
stegosaurus. Fun!

```sh
FROM ubuntu:16.04

RUN apt-get update
RUN apt-get install -y cowsay python3 python3-pip
RUN pip3 install pyjokes

CMD pyjoke | /usr/games/cowsay -f stegosaurus
```

```sh
pyjokes             latest              4c5a96439b92        About a minute ago   440MB
```

## Dockerfile.2

Dockerfile.1 was pretty big. Can we reduce it?

```sh
4c4
< RUN apt-get install -y cowsay python3 python3-pip
---
> RUN apt-get install -y cowsay python3 python3-pip --no-install-recommends
```

```sh
pyjokes             latest              5926b27d9d60        About a minute ago   231MB
```

## Dockerfile.3

Let's rm some more unnecessary things. What happens when you remove things in a new layer from a previous layer?

```sh
4a5
> RUN rm -rf /var/lib/apt/lists/*
```

```sh
pyjokes             latest              2286d8846753        About a minute ago   231MB
```

## Dockerfile.4

Dockerfile.3 removed files but didn't reduce the image size because it was in a
new layer. Combine layers to resolve. Also, `apt-get update` should always be
on the same line as `apt-get install` - otherwise the sources may become out of
date over time.

```sh
3,5c3,5
< RUN apt-get update
< RUN apt-get install -y cowsay python3 python3-pip --no-install-recommends
< RUN rm -rf /var/lib/apt/lists/*
---
> RUN apt-get update && \
>     apt-get install -y cowsay python3 python3-pip --no-install-recommends && \
>     rm -rf /var/lib/apt/lists/*
```

```sh
pyjokes             latest              fbee4f9746d0        About a minute ago   192MB
```

## Dockerfile.5

To really get the image down, try python's alpine linux container. No cowsay :(

```sh
FROM python:3.6-alpine

RUN pip3 install pyjokes

CMD pyjoke
```

```sh
pyjokes             latest              245a62171dc0        32 minutes ago      92.9MB
```

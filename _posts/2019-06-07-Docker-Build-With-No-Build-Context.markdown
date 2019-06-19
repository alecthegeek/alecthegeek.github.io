---
layout: post
title: "Running Docker build with no build context"
date: 2019-06-06 17:02
comments: true
categories: docker
---


NOTE: Updated 20/June/2019 after a talk I gave at [PaperCut Software](https:) on building developer
images. After feedback it was pretty obvious that the material assumed a lot of prior knowledge.
I've added more details about the "classic" file layout for `..image build` before looking at the alternatives.

You can find my slides and demo scripts from that talk [here](https://github.com/PaperCutSoftware/docker-images-for-development)


When using the ``docker image build`` command the last argument is the build context.
This is usually the location of the files needed during the build. For example

* Dockerfile
* [Docker ignore file](https://docs.docker.com/engine/reference/builder/#dockerignore-file)
* Files needed for the image to work. e.g. startup intialisation scripts.

Developers often keep their source in the same directory as well.

So if the current default directory contains all the files in the build context the build is
run as follows:

```
docker image build  -t my-dev-image:0.1 .
```

Note the `.` to specify the current working directory as the build context.

Immediately the novice is presented with additional details about creating the contents of the build
context, how to use a Docker ignore file, and so on. This is in addition to understanding
how to create a Dockerfile, which can be complex in itself.

However the `-` alternative (no build context) is usually not presented until much later, or even never at all.

The `-` context works by reading the docker file
from standard input, for instance

```sh
docker image build -t myTestImage:0.1 - < Dockerfile
```

I really believe that the `-` feature should
be the *first* example novices see because it's much
easier to understand.
Build contexts and other details can come later.

"But you can't create a useful container without adding
files from the build context" I hear you say.
"Piffle!" I reply, "here is an example".

**NOTE**: In order to make the examples self contained
I am using here documents so I don't have to show
a separate dockerfile.

```sh
#!/usr/bin/env sh

docker image build -t demo1:0.1 --build-arg LUA_VERSION=5.1 - <<EOF
FROM alpine:3.9

ARG LUA_VERSION=5.3

LABEL author "Alec Clews <alecclews@gmail.com>"
LABEL description "Linux with Lua"

RUN apk add --no-cache lua${LUA_VERSION}

CMD [ "/usr/bin/lua" ]
EOF

```
or in PowerShell

```powershell
@'
FROM alpine:3.9

ARG LUA_VERSION=5.3

LABEL author "Alec Clews <alecclews@gmail.com>"
LABEL description "Linux with Lua"

RUN apk add --no-cache lua${LUA_VERSION}

CMD [ "/usr/bin/lua" ]
'@ | docker image build -t demo1:0.1 --build-arg LUA_VERSION=5.1 -
```

No matter what the contents of the current directory, no files are transferred to the Docker server
to complete the build.

Run the image with ``docker container run -it --rm demo1:0.1``

Using this approch you can create useful images and introduce a number of build topics.
For example: ``build-arg``, ``LABELS``, ``RUN``, ``CMD``, etc without having to be concerned about the 
files in the build context or the Docker ignore file. The concepts can be confusing
for many novices because they are not familiar with Docker's client server architecture yet.

Even the potentially confusing discussion of the differences between ``ENTRYPOINT`` and ``CMD``
can be demonstrated without using a build context.

```sh
#!/usr/bin/env bash

docker image build -t figlet:1.0 - <<EOF
FROM alpine:3.9

LABEL author "Alec Clews <alecclews@gmail.com>"
LABEL description "ENTRYPOINT vs CMD"

RUN apk add --no-cache figlet

ENTRYPOINT ["/usr/bin/figlet"]
CMD ["Default message from Figlet"]
EOF

echo Now test a few examples

echo docker container run -it --rm figlet:1.0
docker container run -it --rm figlet:1.0

echo docker container run -it --rm figlet:1.0 Hello There!
docker container run -it --rm figlet:1.0 Hello There!

echo docker container run -it --rm figlet:1.0 sh
docker container run -it --rm figlet:1.0 sh

docker image build -t figlet:2.0 - <<EOF
FROM figlet:1.0
ENTRYPOINT []
CMD ["/usr/bin/figlet"]
EOF

echo docker container run -it --rm figlet:2.0 sh -c 'sh --help 2>&1 | head -n 1'
docker container run -it --rm figlet:2.0 sh -c 'sh --help 2>&1 | head -n 1'
```

If you are not sure what you seeing here then I can recommend this handy tutorial
on ``CMD`` and ``ENTRYPOINT``.

https://container.training/intro-selfpaced.yml.html#toc-cmd-and-entrypoint
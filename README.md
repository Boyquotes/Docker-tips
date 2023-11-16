user
sudo chown -R $(id -u):$(id -g) $HOME/.docker

## Dockerfile
```
ARG BASE_IMAGE_NAME=debian:bullseye-slim
# Intermediate build container
FROM $BASE_IMAGE_NAME AS builder

LABEL maintainer="boyquotes <boyquotes@proton.me>"
LABEL name="nvm-dev-env"
LABEL version="latest"

ENV NVM_DIR /usr/local/nvm
ENV NODE_VERSION 4.2.4
ENV NODE_PATH $NVM_DIR/v$NODE_VERSION/lib/node_modules
ENV PATH      $NVM_DIR/versions/node/v$NODE_VERSION/bin:$PATH

COPY --from=composer_upstream --link /composer /usr/bin/composer


=====
FROM python:latest
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
COPY . /app
WORKDIR /app

SHELL ["/bin/bash", "-c"]
# Set the SHELL to bash with pipefail option
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
RUN /bin/bash -c "source /usr/local/bin/virtualenvwrapper.sh \
    && mkvirtualenv myapp \
    && workon myapp \
    && pip install -r /mycode/myapp/requirements.txt"
RUN ["/bin/bash", "-c", "source /usr/local/bin/virtualenvwrapper.sh"]

RUN pip3 install -r requirements.txt
# Set locale
RUN locale-gen en_US.UTF-8

## https://github.com/nvm-sh/nvm/blob/master/Dockerfile
# Add user "nvm" as non-root user
RUN useradd -ms /bin/bash nvm
# Copy and set permission for nvm directory
COPY . /home/nvm/.nvm/
RUN chown nvm:nvm -R "/home/nvm/.nvm"
# Set sudoer for "nvm"
RUN echo 'nvm ALL=(ALL) NOPASSWD: ALL' >> /etc/sudoers
# Switch to user "nvm" from now
USER nvm
# nvm
RUN echo 'export NVM_DIR="$HOME/.nvm"'                                       >> "$HOME/.bashrc"
RUN echo '[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm' >> "$HOME/.bashrc"
RUN echo '[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion" # This loads nvm bash_completion' >> "$HOME/.bashrc"
# nodejs and tools
RUN bash -c 'source $HOME/.nvm/nvm.sh   && \
    nvm install node                    && \
    npm install -g doctoc urchin eclint dockerfile_lint && \
    npm install --prefix "$HOME/.nvm/"'

# Set WORKDIR to nvm directory
WORKDIR /home/nvm/.nvm

EXPOSE 3000
EXPOSE 8080:8080

ENTRYPOINT ["/bin/bash"]
====
# Add 3.7 to the available alternatives
RUN update-alternatives --install /usr/bin/python python /usr/bin/python3.7 1
# Set python3.7 as the default python
RUN update-alternatives --set python /usr/bin/python3.7

ENTRYPOINT [ "python3" ]
ENTRYPOINT ["bash", "--rcfile", "/usr/local/bin/virtualenvwrapper.sh", "-ci"]

CMD [ "app.py" ]
CMD [ "python", "run.py"]
CMD ["tail", "-f", "/dev/null"]

# Prevent dialog during apt install
ENV DEBIAN_FRONTEND noninteractive
## Install build dependencies
RUN set -e && \
	apt-get update && apt-get install --assume-yes --no-install-recommends \
		ca-certificates \
		dirmngr \
		dpkg-dev \
		gcc \
		gnupg \
		libbz2-dev \
		libc6-dev \
		libexpat1-dev \
		libffi-dev \
		liblzma-dev \
		libsqlite3-dev \
		libssl-dev \
		make \
		netbase \
		uuid-dev \
		wget \
		xz-utils \
		zlib1g-dev

# Install apt packages
RUN apt update         && \
    apt upgrade -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold"  && \
    apt install -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold"     \
        coreutils             \
        util-linux            \
        bsdutils              \
        file                  \
        openssl               \
        libssl-dev            \
        locales               \
        ca-certificates       \
        ssh                   \
        wget                  \
        patch                 \
        sudo                  \
        htop                  \
        dstat                 \
        vim                   \
        tmux                  \
        curl                  \
        git                   \
        jq                    \
        zsh                   \
        ksh                   \
        gcc                   \
        g++                   \
        xz-utils              \
        build-essential       \
        bash-completion       && \
    apt-get clean


## FROM
FROM python:3  
FROM python:<version>-slim  
FROM python:<version>-alpine  

FROM python:3.7.5-slim
RUN python -m pip install \
        parse \
        realpython-reader
```

## RUN
`docker run -it --rm --name my-running-script -v "$PWD":/usr/src/myapp -w /usr/src/myapp python:3 python your-daemon-or-script.py`

## MULTI-STAGE
$ docker build --target builder -t alexellis2/href-counter:latest .

You can even use an external image as a stage, it is described in the same docs, so you can do something like this:

COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf

# syntax=docker/dockerfile:1
FROM ubuntu AS base
RUN echo "base"

FROM base AS stage1
RUN echo "stage1"

FROM base AS stage2
RUN echo "stage2"
With BuildKit enabled, building the stage2 target in this Dockerfile means only base and stage2 are processed. There is no dependency on stage1, so it's skipped.

DOCKER_BUILDKIT=0 docker build --no-cache -f Dockerfile --target stage2 .

# syntax=docker/dockerfile:1

FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
---
```
# syntax=docker/dockerfile:1
FROM golang:1.21 as build
WORKDIR /src
COPY <<EOF /src/main.go
package main

import "fmt"

func main() {
  fmt.Println("hello, world")
}
EOF
RUN go build -o /bin/hello ./main.go

FROM scratch
COPY --from=build /bin/hello /bin/hello
CMD ["/bin/hello"]
```

```
# syntax=docker/dockerfile:1

FROM alpine:latest AS builder
RUN apk --no-cache add build-base

FROM builder AS build1
COPY source1.cpp source.cpp
RUN g++ -o /binary source.cpp

FROM builder AS build2
COPY source2.cpp source.cpp
RUN g++ -o /binary source.cpp
```

```
FROM alpine as s1
RUN sleep 10 && echo "s1 done"

FROM alpine as s2
RUN sleep 10 && echo "s2 done"

FROM alpine as s3
RUN sleep 10 && echo "s3 done"
Sequential: docker build . will take around 30 seconds.

Parallel: DOCKER_BUILDKIT=1  docker build . will take around 10 seconds.


nano db_password.txt
PMA_PASSWORD='admin'
MYSQL_ROOT_PASSWORD='admin'
MYSQL_PASSWORD='admin'
docker run --name myadmin -d -e PMA_PASSWORD_FILE=db_password.txt -p 8080:80 phpmyadmin

```




```
apk add nodejs npm
apk add nodejs-current
```
```
docker build --build-arg some_variable_name=a_value
docker run -e "env_var_name=another_value" alpine env

env file
env_var_name=another_value
env_var_name2=yet_another_value

 docker run --env-file=BOB alpine env
 docker run -it -p 8080:8080 rotki_linux bash
 docker exec -it reverent_murdock bash

 docker create --name `container name` --expose 7000 --expose 7001 `image name`
 docker run -it -p 8888-8898:8888-8898
 EXPOSE 8000-8009
The above command tells that the application needs to open 10 ports starting from 8000 to 8009 for communication
In short, running docker run --network host ... will expose all the container ports.
With docker 1.3, there is a new command docker exec. This allows you to enter a running container:

docker exec -it [container-id] bash

docker run --name python311 -it --entrypoint /bin/bash python311
docker build -t rotki-linux . && docker run -d --name rot -p 8080:8080 -p 4242:4242 -it rotki-linux  && docker logs -f rot
docker run -d --name rot -p 8080:8080 -p 4242:4242 -it rotki-linux
docker logs -f rot
docker stop rot && docker rm rot

ADD "https://api.github.com/repos/boyquotes/rotki/commits?per_page=1" sha

HEALTHCHECK --start-period=60s CMD curl -f http://localhost:2019/metrics || exit 1

SYMFONY_VERSION=6.1.* docker compose up -d --wait
STABILITY=dev docker compose up -d --wait

Commit the stopped container to a new image: test_image.
docker commit $CONTAINER_ID test_image
Run the new image in a new container with a shell.
docker run -ti --entrypoint=sh test_image
Run the list file command in the new container.
docker exec --privileged $NEW_CONTAINER_ID ls -1 /var/log

docker export -o dump.tar <container id>
tar -tvf dump.tar

DinD
docker run --privileged -p 2376:2376 -p 4443:443 -p 8000:80 --name dind_docker2 -d docker:dind
# docker run --privileged -p 2376:2376 -p 4443:4443 -p 8000:8000 --name dind_docker -d docker:dind
docker exec -it dind_docker /bin/sh
apk add git curl make nano
git clone https://github.com/dunglas/symfony-docker.git
cd symfony-docker
docker compose build --no-cache
HTTP_PORT=8000 HTTPS_PORT=4443 HTTP3_PORT=4443 docker compose up -d --wait

docker run -d --privileged -p 2376:2376 -p 4443:4443 -p 8000:8000 sf6 /bin/sh

docker container cp <container id>:/etc/nginx/conf.d/default.conf ./


docker inspect dind_docker2 | grep IPAddress
iptables -t nat -A  DOCKER -p tcp --dport 4443 -j DNAT --to-destination 172.17.0.2:4443
iptables -t nat -A  DOCKER -p tcp --dport 8000 -j DNAT --to-destination 172.17.0.2:8000

docker compose up --pull always|never|policy -d --wait

PHP
docker compose exec -it php sh
====

CLEAN

docker ps -aq | xargs docker stop | xargs docker rm

docker rm -f $(docker ps -a -q)
Delete all containers using the following command:
docker rm -f $(docker ps -a -q)
Delete all volumes using the following command:
docker volume rm $(docker volume ls -q)
docker system df -v

Just run these three. No need to remove RUNNING containers.

Cleanup exited processes:

docker rm $(docker ps -q -f status=exited)
Cleanup dangling volumes:

docker volume rm $(docker volume ls -qf dangling=true)
Cleanup dangling images:

docker rmi $(docker images --filter "dangling=true" -q --no-trunc)
docker system prune -a -f --volumes
    -a == removes all unused images
    -f == force
    --volumes == prune volumes.
docker system prune -f --volumes


## VOLUMES
docker volume ls

```



## PYTHON
API :  
https://github.com/docker/docker-py

https://github.com/docker-training/webapp  
https://github.com/codefresh-contrib/python-flask-sample-app  

`docker run -it --rm quay.io/python-devs/ci-image:master`


## PHP
composer install --no-cache --prefer-dist --no-dev --no-autoloader --no-scripts --no-progress


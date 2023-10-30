
## Dockerfile
```
FROM python:latest
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
COPY . /app
WORKDIR /app

SHELL ["/bin/bash", "-c"]
RUN /bin/bash -c "source /usr/local/bin/virtualenvwrapper.sh \
    && mkvirtualenv myapp \
    && workon myapp \
    && pip install -r /mycode/myapp/requirements.txt"
RUN ["/bin/bash", "-c", "source /usr/local/bin/virtualenvwrapper.sh"]

RUN pip3 install -r requirements.txt

ENTRYPOINT [ "python3" ]
ENTRYPOINT ["bash", "--rcfile", "/usr/local/bin/virtualenvwrapper.sh", "-ci"]

CMD [ "app.py" ]
CMD [ "python", "run.py"]
```



## FROM
FROM python:3  
FROM python:<version>-slim  
FROM python:<version>-alpine  
```
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

## PYTHON
`docker run -it --rm quay.io/python-devs/ci-image:master`
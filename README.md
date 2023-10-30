$ docker build --target builder -t alexellis2/href-counter:latest .

You can even use an external image as a stage, it is described in the same docs, so you can do something like this:

COPY --from=nginx:latest /etc/nginx/nginx.conf /nginx.conf
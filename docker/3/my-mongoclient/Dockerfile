FROM alpine:3.9
# alpine 3.9 has mongodb 4.0.5-r0 in community repository
# https://hub.docker.com/_/alpine/
RUN echo 'http://dl-cdn.alpinelinux.org/alpine/v3.9/main' >> /etc/apk/repositories \ 
    && echo 'http://dl-cdn.alpinelinux.org/alpine/v3.9/community' >> /etc/apk/repositories \ 
    && apk update \
    && apk add mongodb
CMD ["/bin/sleep","1d"]

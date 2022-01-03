# Fetch latest changes from Github
FROM alpine/git:latest as repo
RUN git clone -b main --depth 1 https://github.com/mattermost/focalboard.git /focalboard

# Build webapp
FROM node:16.3.0-alpine as nodebuild
WORKDIR /webapp
COPY --from=repo /focalboard/webapp /webapp
RUN export CFLAGS="$CFLAGS -DPNG_ARM_NEON_OPT=0" && npm install --no-optional && npm run pack

# Build Go backend
FROM golang:1.16.5-alpine as gobuild
WORKDIR /go/src/focalboard
COPY --from=repo /focalboard /go/src/focalboard
RUN sed -i "s/GOARCH=amd64/GOARCH=arm64/g" Makefile
RUN make server-linux
RUN mkdir /data

# Build Final image
FROM gcr.io/distroless/base-debian10

WORKDIR /opt/focalboard

COPY --from=gobuild --chown=nobody:nobody /data /data
COPY --from=nodebuild --chown=nobody:nobody /webapp/pack pack/
COPY --from=gobuild --chown=nobody:nobody /go/src/focalboard/bin/linux/focalboard-server bin/
COPY --from=gobuild --chown=nobody:nobody /go/src/focalboard/LICENSE.txt LICENSE.txt
COPY --from=gobuild --chown=nobody:nobody /go/src/focalboard/docker/server_config.json config.json

USER nobody

EXPOSE 8000/tcp 9092/tcp

VOLUME /data

CMD ["/opt/focalboard/bin/focalboard-server"]
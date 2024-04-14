FROM alpine:3.19.1

LABEL org.opencontainers.image.authors="The Automation Rules <theautomationrules@gmail.com>" \
      org.opencontainers.image.url="http://www.theautomationrules.com/" \
      org.opencontainers.image.title="custom-alpine" \
      org.opencontainers.image.description="Docker Custom Alpine Image"

ENV ARCH="arm64"
ENV USER="automation"

RUN apk add --update unzip wget git bash openssh

RUN ["mkdir", "-p", "/home/${USER}"]

WORKDIR /home/${USER}/
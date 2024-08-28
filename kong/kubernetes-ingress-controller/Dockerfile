ARG BASE_REGISTRY=registry1.dsop.io
ARG BASE_IMAGE=ironbank/redhat/ubi/ubi9
ARG BASE_TAG=9.4
FROM ${BASE_REGISTRY}/${BASE_IMAGE}:${BASE_TAG}
ENV KICPATH=/usr/local/kubernetes-ingress-controller-2.12.5

COPY LICENSE /licenses/
COPY LICENSES /licenses/
COPY golang.tar.gz /
COPY kic.tar.gz /

RUN groupadd --system kic && \
    adduser --system kic -g kic -u 1001

RUN tar -xzf golang.tar.gz -C /usr/local \
    && tar -xzf kic.tar.gz -C /usr/local \
    && rm -f /golang.tar.gz \
    && rm -f /kic.tar.gz \
    && rm -f /usr/local/go/src/crypto/tls/testdata/example-key.pem \
    && rm -f /usr/local/go/src/crypto/x509/platform_root_key.pem  \
    && yum install -y -q make \
    && dnf upgrade -y \
    && dnf install -y --nodocs git \
    && dnf clean all \
    && rpm -e --nodeps ncurses-6.2-10.20210508.el9.x86_64 \
    && rm -rf /var/cache/dnf \
    && rm /usr/libexec/openssh/ssh-keysign \
    && mkdir /home/kic \
    && chmod 0750 /home/kic \
    && chgrp kic /home/kic  \
    && rm -rf  /usr/share/doc/perl-IO-Socket-SSL/certs/

ENV GOROOT=/usr/local/go
ENV GOPATH=/go
ENV PATH=$GOPATH/bin:$GOROOT/bin:$PATH

RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" \
    && chmod -R 744 "$GOPATH" \
    && chown -R 1001:1001 "$GOPATH"

WORKDIR /workspace
RUN cp "$KICPATH"/go.mod go.mod
RUN cp "$KICPATH"/go.sum go.sum
RUN mv "$KICPATH"/pkg .
RUN mv "$KICPATH"/internal .
#RUN mv "$KICPATH"/controllers .
RUN go mod download
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 GO111MODULE=on GOEXPERIMENT=boringcrypto go build -o bin/manager -ldflags "-s -w" ./internal/cmd/fips/main.go

WORKDIR /
RUN cp /workspace/bin/manager .
RUN yum update -y
RUN rm -rf /go
RUN rm -rf /workspace
RUN rm -rf "$KICPATH"
RUN rpm -e --nodeps git-2.43.5-1.el9_4.x86_64
RUN rpm -e --nodeps git-core-2.43.5-1.el9_4.x86_64
RUN rpm -e --nodeps git-core-doc-2.43.5-1.el9_4.noarch
RUN rpm -e --nodeps perl-Git-2.43.5-1.el9_4.noarch
RUN yum -y remove make

USER 1001

ENTRYPOINT ["/manager"]

HEALTHCHECK NONE

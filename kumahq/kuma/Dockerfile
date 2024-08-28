ARG BASE_REGISTRY=registry1.dsop.io
ARG BASE_IMAGE=ironbank/redhat/ubi/ubi9
ARG BASE_TAG=9.4
FROM ${BASE_REGISTRY}/${BASE_IMAGE}:${BASE_TAG}
ENV MESHPATH=/kuma-2.8.2

COPY LICENSE /licenses/kuma

ARG TARBALL=kuma-rhel-amd64.tar.gz
COPY ["${TARBALL}", "/opt/"]

RUN tar xvzf /opt/${TARBALL} \
    && useradd -u 1001 -s /bin/bash kong \
    && chown -R 1001:1001 "$MESHPATH" \
    && chmod +x "$MESHPATH/bin/coredns" \
    && chmod +x "$MESHPATH/bin/envoy" \
    && chmod +x "$MESHPATH/bin/kuma-cp" \
    && chmod +x "$MESHPATH/bin/kuma-dp" \
    && chmod 0740 /home/kong/.bashrc \
    && chmod 0740 /home/kong/.bash_profile \
    && chmod +x "$MESHPATH/bin/kumactl" \

    && rm -f ./opt/${TARBALL}

#Both DP and CP ports for Global, Remote and standalone
EXPOSE 5443 5653 5676 5677 5678 5679 5680 5681 5682 5683 5685

WORKDIR $MESHPATH/bin

USER 1001

HEALTHCHECK NONE

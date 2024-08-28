ARG BASE_REGISTRY=registry1.dsop.io
ARG BASE_IMAGE=ironbank/redhat/ubi/ubi9
ARG BASE_TAG=9.4
FROM ${BASE_REGISTRY}/${BASE_IMAGE}:${BASE_TAG}

COPY LICENSE /licenses/kong
COPY signatures/kong.key /opt/kong.key

ARG RPM=kong-rhel8.amd64.rpm
COPY ["${RPM}", "/opt/"]

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

RUN set -ex; \
  yum install -y -q unzip shadow-utils \
  && yum clean all -q \
  && rm -fr /var/cache/yum/* /tmp/yum_save*.yumtx /root/.pki \
  && rpm --import /opt/kong.key \
  && yum deplist /opt/${RPM} | awk '/provider:/ {print $2}' | sort -u | xargs yum --nobest --skip-broken -y install \
  && rpm -i --nofiledigest --nodeps --nodigest /opt/${RPM} \
  && yum clean all \
  && ln -sf /usr/local/openresty/bin/resty /usr/local/bin/resty \
  && ln -sf /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit \
  && ln -sf /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua \
  && ln -sf /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx \
  && chmod go-w /usr/local/bin/kong \
  && chmod go-w /usr/local/bin/kong-health \
  && chmod go-w /usr/local/bin/lua \
  && chmod go-w /usr/local/bin/luajit \
  && chmod go-w /usr/local/bin/luarocks \
  && chmod go-w /usr/local/bin/nginx \
  && chmod go-w /usr/local/bin/resty \
  && chown root:root /usr/local/bin/kong \
  && chown root:root /usr/local/bin/luarocks \
  && chown -R kong:0 /usr/local/kong \
  && chmod +x /usr/local/bin/kong \
  && chmod +x /docker-entrypoint.sh \
  && chmod 0740 /home/kong/.bashrc \
  && chmod 0740 /home/kong/.bash_profile \
  && rm -f ./opt/kongkey \
  && rm -f ./opt/${RPM} \
  && rpm -e --nodeps ncurses-6.2-10.20210508.el9.x86_64 \
  && rpm -e --nodeps qt5-srpm-macros-5.15.9-1.el9.noarch \
  && rm -f /usr/share/doc/perl-Net-SSLeay/examples/server_key.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/client-key.enc \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/client-key.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/proxyca.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server2-key.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-ecc-key.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-key.enc \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-key.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-wildcard.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/sub-server.pem \
  && rm -f /usr/share/doc/perl-IO-Socket-SSL/example/simulate_proxy.pl

USER 1001

#Exposing CP and DP Admin and API ports default and SSL
EXPOSE 8000 8443 8001 8444 8002 8445 8005 8006

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health

CMD ["kong", "docker-start"]

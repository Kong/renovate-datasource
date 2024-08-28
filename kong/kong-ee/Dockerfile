ARG BASE_REGISTRY=registry1.dsop.io
ARG BASE_IMAGE=ironbank/redhat/ubi/ubi8
ARG BASE_TAG=8.10
FROM ${BASE_REGISTRY}/${BASE_IMAGE}:${BASE_TAG}

COPY LICENSE /licenses/kong
RUN mkdir -p /home/kong/.luarocks/share/lua/5.1/xmlhandler/
#COPY libraries/xml2lua.lua /home/kong/.luarocks/share/lua/5.1
#COPY libraries/XmlParser.lua /home/kong/.luarocks/share/lua/5.1
#COPY libraries/xmlhandler/ /home/kong/.luarocks/share/lua/5.1/xmlhandler/
COPY signatures/kong.key /opt/kong.key

ARG RPM=kong-enterprise-edition-rhel9.noarch.rpm
ARG DECK=deck.tar.gz
COPY ["${RPM}", "/opt/"]
COPY ["${DECK}", "/opt/"]

COPY docker-entrypoint.sh /docker-entrypoint.sh
ENTRYPOINT ["/docker-entrypoint.sh"]

RUN set -eux; \
   yum install -y -q unzip shadow-utils  ;\
   yum clean all -q ;\
   mkdir /tmp/deck ;\
   tar -xf /opt/deck.tar.gz -C /tmp/deck ;\
   cp /tmp/deck/deck /usr/local/bin/ ;\
   chown root:root /usr/local/bin/deck ;\
   chmod +x /usr/local/bin/deck ;\
   rm -fr /tmp/deck ;\
   rm -fr /var/cache/yum/* /tmp/yum_save*.yumtx /root/.pki ;\
   rpm --import /opt/kong.key ;\
   yum deplist /opt/${RPM} | awk '/provider:/ {print $2}' | sort -u | xargs yum -y install ;\
   rpm -i --nofiledigest --nodeps --nodigest /opt/${RPM} ;\
   yum clean all ;\
   rm -rf /var/cache/dnf ;\
   ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/luajit ;\
   ln -s /usr/local/openresty/luajit/bin/luajit /usr/local/bin/lua ;\
   ln -s /usr/local/openresty/nginx/sbin/nginx /usr/local/bin/nginx ;\
   chown root:root /usr/local/bin/kong ;\
   chown root:root /usr/local/bin/luarocks ;\
   chown root:root /usr/local/bin/resty ;\
   chmod go-w /usr/local/bin/kong ;\
   chmod go-w /usr/local/bin/lua ;\
   chmod go-w /usr/local/bin/luajit ;\
   chmod go-w /usr/local/bin/luarocks ;\
   chmod go-w /usr/local/bin/nginx ;\
   chmod go-w /usr/local/bin/resty ;\
   chmod -R 0750 /home/kong/ ;\
   chgrp kong /home/kong ;\
   chown -R kong:0 /usr/local/kong ;\
   chmod +x /usr/local/bin/kong ;\
   chmod +x /docker-entrypoint.sh ;\
   rm -f ./opt/kongkey ;\
   rm -f ./opt/${RPM} ;\
   rm -f ./opt/${DECK} ;\
   rpm -e --nodeps ncurses-6.1-10.20180224.el8.x86_64 ;\
   rpm -e --nodeps qt5-srpm-macros-5.15.3-1.el8.noarch ;\
   rm -f /usr/share/doc/perl-Net-SSLeay/examples/server_key.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/client-key.enc ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/client-key.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/proxyca.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server2-key.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-ecc-key.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-key.enc ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-key.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/server-wildcard.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/certs/sub-server.pem ;\
   rm -f /usr/share/doc/perl-IO-Socket-SSL/example/simulate_proxy.pl

USER 1001

ENV KONG_FIPS=on

#Exposing CP, DP, and portal Admin and API ports default and SSL
EXPOSE 8000 8443 8001 8444 8002 8445 8005 8006 8003 8446 8004 8447

STOPSIGNAL SIGQUIT

HEALTHCHECK --interval=10s --timeout=10s --retries=10 CMD kong health

CMD ["kong", "docker-start"]

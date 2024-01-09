FROM sameersbn/bind:9.16.1-20200524

COPY conf/named.conf.default-zones /etc/bind/
COPY conf/named.conf.options /etc/bind/
COPY conf/named.conf.local /etc/bind/

COPY conf/zones/* /etc/bind/zones/

COPY conf/GeoIP.acl /etc/bind/geoip/

EXPOSE 53/udp 53/tcp

CMD ["/usr/sbin/named"]
FROM golang:1.19.2-alpine as unpack
ARG GOPROXY=https://goproxy.cn,direct
WORKDIR /
RUN set -xe \
  && apk add --no-cache curl \
  && go install github.com/urlesistiana/v2dat@latest \
  && curl -LJO https://ghp.ci/https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat \
  && curl -LJO https://ghp.ci/https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat \
  && mkdir dat \
  && v2dat unpack geoip -o dat -f "private" geoip.dat \
  && v2dat unpack geoip -o dat -f "cn" geoip.dat \
  && v2dat unpack geosite -o dat -f "cn" geosite.dat \
  && v2dat unpack geosite -o dat -f "gfw" geosite.dat \
  && v2dat unpack geosite -o dat -f "category-ads-all" geosite.dat \
  && v2dat unpack geosite -o dat -f "geolocation-!cn" geosite.dat

#use multi-state builds
FROM irinesistiana/mosdns:latest
LABEL maintainer="None"

# RUN set -xe \
#   && apk add --no-cache curl \
#   && cd / \
#   && curl -LJO https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat \
#   && curl -LJO https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat \
#   && apk del curl
# # &&  echo '0 2 * * *  wget -O /geoip.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geoip.dat'>>/var/spool/cron/crontabs/root  \
# # &&  echo '0 2 * * *  wget -O /geosite.dat https://github.com/Loyalsoldier/v2ray-rules-dat/releases/latest/download/geosite.dat'>>/var/spool/cron/crontabs/root  \
# # && apk del curl
RUN set -xe \
  && apk add --no-cache logrotate \
  && echo "0 1 * * * /bin/sh -c /usr/sbin/logrotate -l=syslog -d /etc/logrotate.d" >> /var/spool/cron/crontabs/root

#mosdns logrotate
COPY ./mosdns /etc/logrotate.d/mosdns

#mosdns config
COPY config.yaml dat_exec.yaml dns.yaml /etc/mosdns/

# all dat files
COPY --from=unpack /dat /dat

#run command
COPY entrypoint.sh /entrypoint.sh
RUN chmod a+x /entrypoint.sh

VOLUME /etc/mosdns
EXPOSE 53/udp 53/tcp
# CMD /usr/bin/mosdns start --dir /etc/mosdns

ENTRYPOINT [ "/entrypoint.sh" ]

FROM node:16-alpine AS build

ARG CRYPTPAD_VERSION

# for some reason bower needs git to be installed ..
RUN apk add --no-cache curl tar git
# RUN npm install -g bower

USER root

RUN curl -L https://github.com/xwiki-labs/cryptpad/archive/refs/tags/${CRYPTPAD_VERSION}.tar.gz -o cryptpad.tar.gz \
    && tar -xvzf cryptpad.tar.gz \
    && mv cryptpad-${CRYPTPAD_VERSION} cryptpad \
    && rm -f cryptpad.tar.gz \
    && rm -f /cryptpad/config/config.example.js

WORKDIR /cryptpad

# RUN sed -i "s@//httpAddress: '::'@httpAddress: '0.0.0.0'@" /cryptpad/config/config.example.js

COPY cryptpad/config.js /cryptpad/config/config.js
COPY cryptpad/application_config.js /cryptpad/customize/application_config.js

RUN npm install --production \
    && npm install -g bower \
    && bower install --allow-root

FROM node:16-alpine

# RUN npm install -g pm2

RUN addgroup -g 4001 -S cryptpad \
    && adduser -u 4001 -S -D -G cryptpad -H -h /dev/null cryptpad \
    && mkdir -p /var/run/nginx \
    && apk add supervisor nginx openssl zlib pcre \
    && rm -rf /etc/nginx

COPY --from=nginx:1.23.3-alpine /etc/nginx /etc/nginx
COPY cryptpad/nginx/nginx.conf /etc/nginx/conf.d/cryptpad.conf
COPY --chown=root cryptpad/letsencrypt/archive/pad.tchartron.com/ /etc/letsencrypt/archive/pad.tchartron.com/
RUN mkdir -p /etc/letsencrypt/live/pad.tchartron.com \
    && ln -s /etc/letsencrypt/archive/pad.tchartron.com/cert1.pem /etc/letsencrypt/live/pad.tchartron.com/cert.pem \
    && ln -s /etc/letsencrypt/archive/pad.tchartron.com/chain1.pem /etc/letsencrypt/live/pad.tchartron.com/chain.pem \
    && ln -s /etc/letsencrypt/archive/pad.tchartron.com/fullchain1.pem /etc/letsencrypt/live/pad.tchartron.com/fullchain.pem \
    && ln -s /etc/letsencrypt/archive/pad.tchartron.com/privkey1.pem /etc/letsencrypt/live/pad.tchartron.com/privkey.pem

COPY --chown=root cryptpad/letsencrypt/ssl-dhparams.pem /etc/letsencrypt/ssl-dhparams.pem

# RUN openssl dhparam -out /etc/nginx/dhparam.pem 4096

# Disable server tokens
RUN sed -i "/default_type/a \\    server_tokens off;" /etc/nginx/nginx.conf

COPY --from=build --chown=cryptpad /cryptpad /cryptpad

# Copy supervisord conf file
COPY cryptpad/supervisor/supervisord.conf /etc/supervisord.conf

# Copy docker-entrypoint.sh script
# COPY docker-entrypoint.sh /docker-entrypoint.sh

# USER cryptpad
WORKDIR /cryptpad

RUN mkdir -p blob block customize data datastore \
    && chown cryptpad:cryptpad blob block customize data datastore

VOLUME /cryptpad/blob
VOLUME /cryptpad/block
VOLUME /cryptpad/customize
VOLUME /cryptpad/data
VOLUME /cryptpad/datastore

EXPOSE 80 443

# ENTRYPOINT ["/bin/sh", "/docker-entrypoint.sh"]

CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisord.conf"]

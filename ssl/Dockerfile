ARG NGINX_VERSION

FROM nginx:${NGINX_VERSION:-1.23.3-alpine}

RUN apk add --update python3 py3-pip certbot \
    && pip3 install pip --upgrade \
    && pip3 install certbot-nginx

RUN mkdir /etc/letsencrypt

COPY ssl/ssl.conf.template /etc/nginx/templates/ssl.conf.template

VOLUME /etc/letsencrypt

# ENTRYPOINT ["/bin/sh"]

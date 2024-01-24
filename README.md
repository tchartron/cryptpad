# Pad

## Steps
```shell
cp .env.example .env
docker build -t nginx-ssl:0.0.1 -f ssl/Dockerfile .
docker run --rm -it -e SERVER_NAME="pad.tchartron.com pad-sandbox.tchartron.com" -p 80:80/tcp --name nginx-ssl -v $(pwd)/cryptpad/letsencrypt:/etc/letsencrypt nginx-ssl:0.0.1
docker exec -it nginx-ssl certbot --nginx -d pad.tchartron.com -d pad-sandbox.tchartron.com
docker cp nginx-ssl:/etc/nginx/conf.d/ssl.conf ./ssl/ssl.certbot.conf

sudo chown -R debian:debian cryptpad/letsencrypt
docker compose build cryptpad --no-cache

chown -R 4001:4001 volumes
```

# immich-app nginx

Nginx is used for some or all of the following:

- SSL offloading
- Block traffic if not routed via CDN first
- Caching
- Rate limit
- IP filtering
- Country restricitons (using CDN's provided headers)
- Rewrites
- Error handling

## Dockerfile

We start from official latest Nginx container
- Install certbot
- Copy our config with our custom `server {}` block

## Host cronjob

To renew local LE certificate we need to set up a cron job in the host running docker
```
sudo cp host-cron.d-immich-nginx-certbot /etc/cron.d
```

## Certbot initial set up:

Nginx uses a self-signed cert, once issue a valid cert update `immich.conf` cert paths.

Example if initial issue:
```
DOMAIN=photos.example.com
EMAIL=myname@example.com
WEBROOT=/usr/share/nginx/html

certbot certonly \
  -d $DOMAIN \
  --webroot -w $WEBROOT \
  --email $EMAIL \
  --agree-tos \
  -n
```

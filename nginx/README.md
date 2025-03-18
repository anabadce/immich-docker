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

Initially Nginx uses a self-signed cert, great for local tests but not good for public Internet traffic.

You should issue a valid SSL certificate that validates the public domain that points to your home.
- Like `home.example.com`, this will only be visible and used by servers to protect traffic from AWS Cloudfront to your origin (home).
- Not `photos.example.com` if that is the subdomain you want users to use, that will be issued and managed by AWS Cloudfront.

Once issued a valid cert update `immich.conf` cert paths.

Example of initial issue:
```
DOMAIN=home.example.com
EMAIL=myname@example.com
WEBROOT=/usr/share/nginx/html

certbot certonly \
  -d $DOMAIN \
  --webroot -w $WEBROOT \
  --email $EMAIL \
  --agree-tos \
  -n
```

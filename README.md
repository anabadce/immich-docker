# Immich App + Cloudfront + Nginx

Example implementaion of Immich server running at home.

Taken from here: https://github.com/immich-app/immich/blob/main/docker/docker-compose.yml

Ajusted to use Cloudfront + Nginx to expose it to the Internet.

👤 (Users)  ➡️ (Cloudfront) ➡️ (Nginx) ➡️ (Immich)

Example:
- Users browse https://photos.example.com -> that points to Cloudfront
- Cloudfront forwards traffic to your home server -> https://home.example.com

## Usage
 
### 1. Create and configure `.env` file from template
- Must change `TZ` and `DB_PASSWORD`

### 2. Create and Configure `nginx/.conf.d/immich.conf` file from template
- Must change:
  - L4: `# SECRET KEY #` that must be he same configured later in Cloudfront
  - L9-11: to restric traffic to desired countries only
  - L16, L32: to allow your local IP range
  - L67: replace `photos.example.com` with the public DNS domain you own and want to use

### 3. Prepare file with trusted proxies and issue a self-signed cert
- Go to [nginx/ssl](./nginx/ssl) and issue `self-signed.crt` and `self-signed.key` using example provided
- Go back to repo root and run script that generates IP list of Cloudfront trusted proxies
```
./update-nginx-proxy-list.sh
```

### 4. Build and run Immich app using `docker`
```
docker compose up -d --build
```
- You can now browse and configure Immich locally:
  - Using Nginx: https://localhost
  - Direct: http://localhost:2283

### 5. Issue a real and signed SSL certificate using Let's Encrypt
- Follow instructions in [nginx](./nginx)
- You will need:
  - A public domain pointing to your server, example: `home.example.com` -> Your home IP
    - Note: this is different domain than what users will use.
  - Port forward configured in your router, port 80 and 443 -> your local server

### 6. Update Nginx to use new certificate
- Cert should be in folder `./letsencrypt_data`
- Update `conf.d/immich.conf` L23-26 so Nginx uses real certificate instead of self-signed.
```
docker compose up -d --build nginx
```
- You can now test browsing https://home.example.com
  - Note: you should see `Nope.`
    - This is because this is only meant to work for traffic coming from Cloudfront.

### 7. Configure Cloudfront in your AWS account
- You want to configure `https://photos.example.com`
- So it forwards to `https://home.example.com`
- Follow instructions in [cloudfront](./cloudfront)

## Notes

- http port 80 and https on port 443 are handled by Nginx
  - By default Nginx uses a self signed certificate
  - You must can issue a valid cert to use Cloudfront
  - Nginx to helps with security, for example restrict countries (recommended)
  - Nginx rejects traffic not passing via Cloudfront
- Immich server runs within docker private network on http port 2283 
- AWS Cloudfront is `FREE*` within limits
  - AWS injects headers that report client info, like Country, used for IP filtering
  - AWS WAF recommended (but that costs extra)
- Alternetively Cloudflare is also free (within limits) but it breaks uploads bigger than 100 MB.

## Final notes

- See logs
```
docker compose logs --follow
```
- As mentioned in [nginx](./nginx) you should set up a cron job to automatically renew your SSL certificate that Nginx uses. Cloudfront renews automatically.

- Understand that exposing your local server and your local photos to the public Internet is risky and you accept all responsability.

## TODO

- Automate update of trusted proxy list
- Automate Immich update process
  - Postgress and Redis image versions are fixed
- Manual update
```
docker compose pull
docker compose down
docker compose up -d --build
```

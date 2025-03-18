## Cloudfront

Recommended to use a CDN like Cloudfront to protect all traffic.

Assuming you have a custom domain and DNS access to it.

- FREE: Issue an SSL certificate using ACM (AWS Certificate Manager)
  - Ensure it is in US East (N. Virginia) Region (us-east-1).
- FREE: Create Cloudfront distribution
  - Alternate domain names must contain your chosen domain and the certificate issued
  - Origin:
    - HTTPS Only
    - Custom header name like `x-home-key` = <some private passphrase>
      - In your local Nginx `nginx/conf.d/immich.conf` update "# SECRET KEY #" so only requests with this key are accepted.
      - Also adjust what countries are allowed (recommended)
      - This is so attackers don't bypass Cloudfront
    - Origin Shield: off
  - Delete all Behaviours but the `Default (*)`
  - Default Behaviour:
    - Viewer protocol policy: Redirect HTTP to HTTPS
    - Legacy cache settings:
      - Headers: all
      - Query strings: all
      - Cookies: all
- NOT Free: enable WAF if you want extra security

## TODO
Write Terraform/Cloudformation/awscli version of this


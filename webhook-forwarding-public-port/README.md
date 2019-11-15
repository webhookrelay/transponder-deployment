
# Transponder for webhook forwarding setup with single public endpoint for incoming webhooks

This example has the same setup as `webhook-forwarding` with one minor difference - there's a `public-endpoint` NGINX based container that allows only incoming webhook traffic into your network.

Update nginx/nginx.conf file to use the same Transponder HTTP port as you have set in the .env file (replace 9301 with 80 or any other port that you chose for Transponder):

```
listen 9999;

location /v1/webhooks/ {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_pass http://transponder:9301;
}
```


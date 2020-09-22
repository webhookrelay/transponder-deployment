
# Transponder for webhook forwarding setup


## Configuration

1. Create a new file `.env`
2. Copy & paste contents of `.env.example` file to `.env` and change the details such as admin username, password API key (key has to remain UUID format) and secret.

### No TLS (when your own firewall/load balancer does termination)

If you don't need TLS in Transponder:

- Ensure that environment variables `CERT_PATH`, `CERT_KEY_PATH` and `MANAGED_DOMAINS` aren't set. Either remove them from the `.env` file or edit the `docker-compose.yaml` to unset them.
- Update `healthcheck` section in the docker-compose.yaml to use http:// instead of https://
- Set `RELAY_REQUIRE_TLS=false` in the agent to disable TLS for GRPC connections. 

Alternatively, if you do need encryption for the agent and you are doing TLS termination in front of the Transponder, you can use `--ws` flag when running forward command:

```bash
relay forward --ws -b my-bucket https://bin.webhookrelay.com/v1/webhooks/d1ea0a51-f317-4e8d-a641-067e96a46bc3 
```

### TLS Configuration

For TLS configuration you can choose between self-signed certificates and the ones provided by [Let's Encrypt](https://letsencrypt.org/). 

> For production instance, set CA_URL=https://acme-v02.api.letsencrypt.org/directory in your .env file. Although it's recommended to first try out your setup with staging CA so you don't hit Let's Encrypt rate limits.

#### Let's Encrypt certificates (TLS-ALPN challenge)

TLS-ALPN challenge is nice to use with webhook forwarding because you don't need a wildcard cert and this method doesn't require 3rd credentials from a DNS provider. Transponder uses this method by default, so just set this environment variable:

```
MANAGED_DOMAINS=your-domain.com
```

Your server must be reachable from the Internet (by Let's Encrypt server).

##### Using DNS challenge

It is recommended to use DNS challenge when you need a wildcard cert **or** your server is not reachable from the public Internet. Transponder supports Cloudflare as a DNS challenge provider. To use it instead of the TLS-ALPN challenge, set these additional variables:

```
CLOUDFLARE_EMAIL=your-cloudflare-account-email
CLOUDFLARE_API_KEY=your-cloudflare-api-key
```

This will ensure that during boot, Transponder will retrieve certificates for your server.

#### Self-signed certificates

Get your certificates and place them into `data/` directory next to this docker-compose.yaml file. Then, set these environment variables in the `.env` file:

```
CERT_PATH=./data/your-domain.pem
CERT_KEY_PATH=./data/your-domain-key.pem
```

## Starting the server

To start the server: 

```bash
docker-compose up -d
```

## Connecting with relay client

You will need to specify several details for the relay client so it can connect to this custom webhook server. Create a new file called `transponder.sh` and specify these variables:

```bash
export RELAY_API_ADDRESS=http://your-server-domain:9301
export RELAY_KEY=cbcf9ee2-7fd2-4e2c-aa59-9c3fc441fdd4
export RELAY_SECRET=change-me
export RELAY_GRPC_API_ADDRESS=your-server-domain:9302    
```

Now, source them:

```bash
source transponder.sh
```

To start using your new Transponder server:

```
relay forward -b my-bucket http://localhost:8080
```

## Accessing admin dashboard

By default, admin dashboard can be accessed on port 9300 (https://your-server-domain:9300). 

# Transponder with self-signed certs example

1. Download and install [mkcert](https://github.com/FiloSottile/mkcert)
2. Generate certs:

```bash
cd self-signed-certs/certs
mkcert example.com 
```

3. Create `.env` file from `.env.example` and add desired configuration 

4. Now, start it: 

```bash
docker-compose up -d
```

# Connecting to a transponder that has self-signed certs 

Create a tunnel (we will be using [xip.io](http://xip.io/) for simplicity):

```bash
export RELAY_API_ADDRESS=https://localhost:9300
export RELAY_TUNNEL_API_ADDRESS=localhost:9800
export RELAY_INSECURE=true
export RELAY_KEY=your-admin-key
export RELAY_SECRET=your-admin-secret

relay tunnel create mytunnel --destination --host mysite.127.0.0.1.xip.io :4000
```

Now, start the tunnelling container:

```bash
docker run --name relayd --net host --restart always \
 -e RELAY_API_ADDRESS=https://localhost:9300 -e RELAY_TUNNEL_API_ADDRESS=localhost:9800 -e RELAY_INSECURE=true \
 webhookrelay/webhookrelayd:1.15.3 --mode tunnel -t my-tunnel -k ADMIN_KEY -s ADMIN_SECRET
```

Now, if you visit https://mysite.127.0.0.1.xip.io:9802/ on your browser, it will route through a local tunnel to http://localhost:4000 web server.
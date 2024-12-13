<h1>
  <p align="center" width="100%">
    Socket-Proxy
  </p> 
</h1>

<h2> 
  <p align="center" width="100%">
    A container to add a security layer to the Docker socket</br>
  </p>
</h2>

<h3>
  <p align="left" width="100%">
    Thanks to the <a href="https://github.com/Tecnativa">Tecnativa</a> team's image: <a href="https://github.com/Tecnativa/docker-socket-proxy">docker-socket-proxy</a>
  </p>
</h3>

[![Static Badge](https://img.shields.io/badge/lang-%F0%9F%87%AA%F0%9F%87%B8_es-blue?style=plastic)](README.es.md)

## Structure:

    socket-proxy/
     ├─ docker-compose.yml              → docker file
     ├─ .env                            → environment variables
     └─ haproxy.cfg.template (Opcional) → internal config (advanced use)

### Important! `haproxy.cfg.template`

The use of `haproxy.cfg.template` is **completely optional**, but if you are going to use it **don't** remove `.template` from the filename. Although it may seem that it's only an example, actually it's the real file that's used by the container.

It can be used for example to fit the timeouts or change the internal listening port, the latter used in very specific cases.

### Other observations (complete comments within each file):

```dockerfile
----
    privileged: true  ← Necessary
    ports:
      - 127.0.0.1:2375:2375  ← This port must be only exposed to the internal network
----
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - $DOCKERDIR/socket-proxy/haproxy.cfg.template:/usr/local/etc/haproxy/haproxy.cfg.template
                      ↖ Comment or delete if not used
----
```
>

## Container start

```bash
docker compose up -d     → start socket-proxy in background

docker logs socket-proxy -f   → examine the log to check if there is any issue (CTRL+c for exit)
```
</br>

## Ready! We now have an extra layer of security on our Docker socket.

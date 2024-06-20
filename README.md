# ns8-postgresql

PostgreSQL is a powerful, open source object-relational database system with over 35 years of active development that has earned it a strong reputation for reliability, feature robustness, and performance.

pgAdmin is the most popular and feature rich Open Source administration and development platform for PostgreSQL, the most advanced Open Source database in the world.

# pgadmin documentation
- https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html

## Install

https://www.pgadmin.org/docs/pgadmin4/latest/container_deployment.html

Instantiate the module with:

    add-module ghcr.io/nethserver/postgresql:latest 1

The output of the command will return the instance name.
Output example:

    {"module_id": "postgresql1", "image_name": "postgresql", "image_url": "ghcr.io/nethserver/postgresql:latest"}

## Configure

Let's assume that the mattermost instance is named `postgresql1`.

Launch `configure-module`, by setting the following parameters:
- `host`: a fully qualified domain name for the pgadmin application
- `http2https`: enable or disable HTTP to HTTPS redirection (true/false)
- `lets_encrypt`: enable or disable Let's Encrypt certificate (true/false)


Example:

```
api-cli run configure-module --agent module/postgresql1 --data - <<EOF
{
  "host": "postgresql.domain.com",
  "http2https": true,
  "lets_encrypt": false
}
EOF
```

The above command will:
- start and configure the postgresql instance
- configure a virtual host for trafik to access the instance


## default credential 

pgadmin needs a default credential to login: `admin@nethserver.org` `Nethesis,1234` the URL is at the `host` property

## connect to database

1 - run locally for maintenance database

    runagent -m postgresql1
    podman exec -ti postgresql-app psql -U postgres


2 - access inside the cluster via the network

```
psql -h IP_of_Node -U postgres -d postgres -p ${TCP_PORT_PGSQL}
```

The password of postgres user can be found inside a secret file `/home/postgresql1/.config/state/secrets/passwords.env`

`${TCP_PORT_PGSQL} `is set inside the environment of the module

`IP_of_Node` is the IP running the container, it must be the internal wiregard IP for example 10.5.4.1, the port is not opened in the firewall

## Get the configuration
You can retrieve the configuration with

```
api-cli run get-configuration --agent module/postgresql1
```

## Uninstall

To uninstall the instance:

    remove-module --no-preserve postgresql1

## Smarthost setting discovery

Some configuration settings, like the smarthost setup, are not part of the
`configure-module` action input: they are discovered by looking at some
Redis keys.  To ensure the module is always up-to-date with the
centralized [smarthost
setup](https://nethserver.github.io/ns8-core/core/smarthost/) every time
postgresql starts, the command `bin/discover-smarthost` runs and refreshes
the `state/smarthost.env` file with fresh values from Redis.

Furthermore if smarthost setup is changed when postgresql is already
running, the event handler `events/smarthost-changed/10reload_services`
restarts the main module service.

See also the `systemd/user/postgresql.service` file.

This setting discovery is just an example to understand how the module is
expected to work: it can be rewritten or discarded completely.

## Debug

some CLI are needed to debug

- The module runs under an agent that initiate a lot of environment variables (in /home/postgresql1/.config/state), it could be nice to verify them
on the root terminal

    `runagent -m postgresql1 env`

- you can become runagent for testing scripts and initiate all environment variables
  
    `runagent -m postgresql1`

 the path become :
```
    echo $PATH
    /home/postgresql1/.config/bin:/usr/local/agent/pyenv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/usr/
```

- if you want to debug a container or see environment inside
 `runagent -m postgresql1`
 ```
podman ps
CONTAINER ID  IMAGE                                      COMMAND               CREATED        STATUS        PORTS                    NAMES
d292c6ff28e9  localhost/podman-pause:4.6.1-1702418000                          9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  80b8de25945f-infra
d8df02bf6f4a  docker.io/library/mariadb:10.11.5          --character-set-s...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  mariadb-app
9e58e5bd676f  docker.io/library/nginx:stable-alpine3.17  nginx -g daemon o...  9 minutes ago  Up 9 minutes  127.0.0.1:20015->80/tcp  postgresql-app
```

you can see what environment variable is inside the container
```
podman exec  postgresql-app env
PG_MAJOR=14
POSTGRES_USER=postgres
TCP_PORT_PGSQL=20040
container=podman
PGADMIN4_IMAGE=docker.io/dpage/pgadmin4:8.6
TRAEFIK_HOST=p3.rocky9-3.org
TCP_PORT_PGADMIN=20041
IMAGE_REOPODIGEST=ghcr.io/nethserver/postgresql@sha256:7214285985f1b83a24349b734e492b39d32627a818a71a71e53ad2f611602904
IMAGE_DIGEST=sha256:7214285985f1b83a24349b734e492b39d32627a818a71a71e53ad2f611602904
PGDATA=/var/lib/postgresql/data
TCP_PORTS_RANGE=20040-20041
GOSU_VERSION=1.17
TRAEFIK_HTTP2HTTPS=False
IMAGE_ID=0697feb0d5ae91dd8aeecfd4ec3cc686ed2a24e8b02a875715898dddfe17ab28
TCP_PORTS=20040,20041
LANG=en_US.utf8
MODULE_ID=postgresql3
NODE_ID=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/lib/postgresql/14/bin
IMAGE_URL=ghcr.io/nethserver/postgresql:opennetwork
TRAEFIK_LETS_ENCRYPT=False
MODULE_UUID=631248ae-6296-45c9-84d7-a981fb269dc1
TCP_PORT=20040
POSTGRES_PASSWORD=d4079c78337e27abd9b200458a46834dbf205218
POSTGRES_IMAGE=docker.io/postgres:14.12-bookworm
PG_VERSION=14.12-1.pgdg120+1
TERM=xterm
HOME=/root
```

you can run a shell inside the container

```
podman exec -ti   postgresql-app sh
/ # 
```
## Testing

Test the module using the `test-module.sh` script:


    ./test-module.sh <NODE_ADDR> ghcr.io/nethserver/postgresql:latest

The tests are made using [Robot Framework](https://robotframework.org/)

## UI translation

Translated with [Weblate](https://hosted.weblate.org/projects/ns8/).

To setup the translation process:

- add [GitHub Weblate app](https://docs.weblate.org/en/latest/admin/continuous.html#github-setup) to your repository
- add your repository to [hosted.weblate.org]((https://hosted.weblate.org) or ask a NethServer developer to add it to ns8 Weblate project

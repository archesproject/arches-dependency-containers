# Arches dependency containers

Provides dependency services by Arches version to allow developers to run projects on multiple versions of Arches simultaneously

## `arches/` (current)

One stack with several versions of each dependency on the `arches-dev` Docker network. Start only what a project needs.

| Service          | Container               | Host port (127.0.0.1) | Use for          |
|------------------|-------------------------|-----------------------|------------------|
| `postgres16`     | `arches-postgres16`     | 5416                  | older projects   |
| `postgres18`     | `arches-postgres18`     | 5418                  | newer projects   |
| `elasticsearch8` | `arches-elasticsearch8` | 9208                  | older projects   |
| `elasticsearch9` | `arches-elasticsearch9` | 9209                  | newer Arches (requires ES9) |
| `pgadmin`        | `arches-pgadmin`        | 5050                  | http://localhost:5050 |

Inside the network, use the container name and the default port (5432 / 9200).

### Setup

```bash
cd arches
cp .env.example .env   # set ELASTIC_PASSWORD
docker compose up -d postgres18 elasticsearch9 pgadmin
```

- Postgres: user `postgres`, password `postgis`; `template_postgis` is created on first start.
- Elasticsearch: user `elastic`, password from `.env`. Security is on; SSL is off. Use the same password as your legacy 7-0 ES so existing env files keep working. `ELASTIC_PASSWORD` only applies when a data volume is first created.
- pgAdmin runs in desktop mode (no login) with both Postgres servers preregistered. It's bound to localhost only.
- Disk-watermark checks are disabled in ES so it still starts on nearly full dev disks.

### Using with arches-docker

`dev-env/` has `env_file.82.env.sample` (PG18 + ES9) and `env_file.76.env.sample` (PG16 + ES8). Copy one to arches-docker's `scaffold/templates/.runtime/development/env_file.<name>.env` (or `$ARCHES_DEV_ENV_DIR`), set `ARCHES_ESPASSWORD`, then scaffold:

```bash
scaffold/init-project.sh --dev 82 --name myproject --output ../myproject
```

The env file sets `DOCKER_NETWORK=arches-dev`, which arches-docker's `make` picks up.

### Moving an existing project from 7-0

1. Point the project at the new stack: set `PGHOST`/`ARCHES_ESHOST` and `DOCKER_NETWORK=arches-dev` in `.runtime/development/env_file.env`, or re-scaffold with `--dev <name>`.
2. Copy the database: `docker exec postgres14-3_arches7-0 pg_dump -U postgres -Fc <db> > db.dump`, then `docker exec -i arches-postgres18 pg_restore -U postgres -d postgres --create < db.dump`.
3. Rebuild the search indexes against the new ES: `make reindex`.

## Legacy stacks

`arches-6-1`, `arches-6-2` and `arches-7-0` are kept as-is for existing setups.

### Used Ports

Arches 6.1.x

    Postgres 13.3.2:
        - 5432
    CouchDB 2.1.1:
        - 5984
    ElasticSearch 7.9.2:
        - 9200
        - 9300
    RabbitMQ 3.x
        - 5672
        - 15672 (admin panel)

Arches 6.2.x

    Postgres 14.3.2:
        - 5433
    CouchDB 2.1.1:
        - 5985
    ElasticSearch 7.9.2:
        - 9201
        - 9301
    RabbitMQ 3.x
        - 5673
        - 15673 (admin panel)

Arches 7.0.x

    Postgres 14.3.2:
        - 5434
    ElasticSearch 8.3.1:
        - 9202
        - 9302
    RabbitMQ 3.x
        - 5674
        - 15674 (admin panel)

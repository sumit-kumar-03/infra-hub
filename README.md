# infra-hub

Shared infrastructure services (databases, queues, etc.) available to all local projects.

Instead of each service repo bundling and managing its own copies of backing
infrastructure (databases, caches, message brokers, admin UIs), those pieces
live here as a single Docker Compose stack that other project repos attach to
over a shared external Docker network (`infra-net`). Container, volume, and
network names are intentionally generic (`infra-mongodb`, not
`cve-trove-mongodb`) since these services are shared, not owned by any one
project.

## Services

Originally moved out of `cve-trove`'s own compose stack:

- `infra-mongodb` — MongoDB, port `27017`, data in the named volume
  `infra-mongodb-data`.
- `infra-mongo-express` — web UI for the above, port `8081`.
- `infra-rabbitmq` — RabbitMQ (management plugin), ports `5672`/`15672`,
  data in the named volume `infra-rabbitmq-data`.

All attach to the `infra-net` Docker network, owned/created here. Any
consumer (e.g. `cve-trove-worker`) joins `infra-net` as an external network
and resolves these services by hostname (`infra-mongodb`, `infra-rabbitmq`).

## Usage

```bash
cp .env.example .env   # fill in real credentials
docker compose up -d
```

## Adding another service's infra later

1. Add its service block(s) to `docker-compose.yml`.
2. Attach it to `infra-net` so other repos' compose stacks can reach it by
   name.
3. Document required env vars in `.env.example`.
4. Name containers/volumes generically (by function, not by the first
   consumer), e.g. `infra-<service>`.

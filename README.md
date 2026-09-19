# infra-hub

Shared infrastructure services (databases, queues, etc.) available to all local projects.

Instead of each service repo bundling and managing its own copies of backing
infrastructure (databases, caches, message brokers, admin UIs), those pieces
live here as a single Docker Compose stack that other project repos attach to
over a shared external Docker network. Container names are intentionally
generic (`infra-mongodb`, not `cve-trove-mongodb`) since these services are
shared, not owned by any one project.

## Phase 1: MongoDB

Moved out of `cve-trove`'s own compose stack. Services:

- `infra-mongodb` — MongoDB, port `27017`, data in the pre-existing named
  volume `cve_trove_mongodb_data` (carried over as-is, no data migration).
- `infra-mongo-express` — web UI for the above, port `8081`.

Both attach to the pre-existing external network `cve_trove_network`, so any
consumer (e.g. `cve-trove-worker`) can resolve the database at the hostname
`infra-mongodb` as long as it's on that same network.

## Usage

```bash
cp .env.example .env   # fill in real credentials
docker compose up -d
```

## Adding another service's infra later

1. Add its service block(s) to `docker-compose.yml`.
2. If it needs to be reachable by name from another repo's compose stack,
   put both on a shared external Docker network (see `cve_trove_network`
   above for the pattern).
3. Document required env vars in `.env.example`.
4. Name containers generically (by function, not by the first consumer),
   e.g. `infra-<service>`.

# Athena deploy

Deployment files for [Athena](https://athena-ai.fi) - a self-hosted home automation platform.
Each `docker-compose-*.yml` here is an independent stack; none of them auto-discover, since only
one file per folder can use Docker Compose's own default name (`docker-compose.yml`). Always pass
`-f` explicitly.

## Athena itself

```
cp .env.example .env    # fill in DB_PASSWORD/DB_ROOT_PASSWORD at minimum
docker compose -f docker-compose-athena.yml up -d
```

Split deploy, no ESPHome dashboard bundled - see `docker-compose-athena.yml`'s own top comment for
running ESPHome's own compiles on a separate machine instead, and that file's own comments
throughout for every other setting (self-update, MQTT, BLE, the Zigbee EZSP USB passthrough and
its hot-plug option, ...).

## Sidecars

Each of these only needs to be running somewhere Athena can reach it over the network - not
necessarily this same host, and not necessarily bundled with Athena's own stack at all. Typically
run on whichever machine the relevant USB stick is physically plugged into.

| File | What it runs | Athena integration |
|---|---|---|
| `docker-compose-zwave.yml` | zwave-js-ui (bundles zwave-js-server) | Z-Wave |
| `docker-compose-deconz.yml` | deCONZ (Phoscon) | deCONZ |
| `docker-compose-zigbee2mqtt.yml` | Zigbee2MQTT | MQTT (Home Assistant discovery convention - no dedicated integration needed) |

Each has its own `.env.<name>.example` - copy to `.env` before running it, or pass
`--env-file .env.<name>` explicitly if running more than one of these (or this one alongside
`docker-compose-athena.yml`) from this same folder, since only one bare `.env` auto-loads per
`docker compose` invocation. `docker-compose-zigbee2mqtt.yml` also needs
`configuration.yaml.example` copied to `./data/configuration.yaml` before its first start.

All three sidecars document two ways to pass their USB stick through - a simple pinned-device
option (the default) and a real hot-plug option (survives an unplug/replug or a stick swap with no
container restart) - see each file's own comments for the exact tradeoff and lines to uncomment.

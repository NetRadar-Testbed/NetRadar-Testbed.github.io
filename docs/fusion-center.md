# 2. Fusion center

The fusion center runs the MQTT broker, time-sync ticks, self-calibration, the
dashboard and a PostgreSQL database as Docker containers
([NetRadar-FusionCenter](https://github.com/NetRadar-Testbed/NetRadar-FusionCenter)).

## Requirements

- Linux (x86-64 or ARM64) with [Docker Engine and the Compose plugin](https://docs.docker.com/engine/install/)
- Tailscale, joined to your tailnet ([step 1](tailscale.md))
- `git`, `openssl` (or `python3`)

## Install

```bash
git clone https://github.com/NetRadar-Testbed/NetRadar-FusionCenter.git
cd NetRadar-FusionCenter

./setup.sh --nodes 3          # or: ./setup.sh --nodes front,left,right
docker compose up -d --build
sudo ./time-sync/chrony-setup.sh master
```

`setup.sh`:

- finds this machine's Tailscale IP (the broker and dashboard listen **only**
  there) and its Tailscale name (what nodes connect to);
- generates random passwords for PostgreSQL, for the fusion center's own MQTT
  user, and **one per node**;
- writes them to `.env`, `nanomq/auth/` and `node-configs/` — all git-ignored and
  readable only by you;
- prints one block per node, like this:

```text
# ---- node 'node1': run ./configure.sh on that node and paste these lines, then Ctrl-D ----
MQTT_HOST=netradar-fc
MQTT_PORT=1883
NODE_ID=node1
MQTT_USERNAME=node1
MQTT_PASSWORD=<48 random hex characters>
# ------------------------------------------------------------------------------
```

You paste that block into the node's `./configure.sh` in the
[next step](node.md). Treat it like a password.

!!! warning "Compose refuses to start without setup.sh"
    The stack has no default passwords. If you run `docker compose up` before
    `./setup.sh`, Compose stops with "run ./setup.sh first". That is intended.

## Check it is running

```bash
docker compose ps          # all services "running" / "healthy"
docker compose logs -f     # follow the logs
```

Open the dashboard from any device on your tailnet:
`http://<fusion-center-name>:5002` (e.g. `http://netradar-fc:5002`). See
[Dashboard](dashboard.md) for what each control does.

## Managing nodes

| Task | Command |
| :--- | :--- |
| Show a node's block again | `./setup.sh --print-node node2` |
| Add a node | `./setup.sh --add-node node4` then `docker compose restart nanomq` |
| Regenerate **all** passwords | `./setup.sh --force`, restart the stack, reconfigure every node |
| Different listen address/name | `./setup.sh --bind-ip <IP> --host <name>` |

## Updating, stopping, data

```bash
git pull && docker compose up -d --build   # update
docker compose down                        # stop (data is kept)
docker compose exec postgres psql -U netradar -d netradar   # query the database
```

Recorded data lives in the Docker volume `pgdata`. The tables are described in
the [repository README](https://github.com/NetRadar-Testbed/NetRadar-FusionCenter#database).

**Next:** [set up the radar nodes](node.md).

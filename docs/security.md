# Security

NetRadar controls real hardware, so treat your testbed's network as private.

## What NetRadar does by default

- **No default passwords.** `setup.sh` generates random passwords on the fusion
  center; Compose refuses to start until it has run. Every node has its own MQTT
  password.
- **Secrets stay out of git.** `.env`, `nanomq/auth/` and `node-configs/` on the
  fusion center, and `.env` on nodes, are git-ignored and readable only by their
  owner.
- **Minimal exposure.** The MQTT broker (1883) and dashboard (5002) listen only
  on the fusion center's Tailscale address; PostgreSQL only on `127.0.0.1`. The
  node web pages (5001, 5003) listen on the node's Tailscale address.
- **Encrypted transport.** Tailscale (WireGuard) encrypts all traffic between
  devices. The broker itself does not use TLS.

## What you should do

- Use a tailnet that only you and your collaborators control. Remove devices you
  no longer use.
- Disable key expiry only for your NetRadar devices, and use short-lived,
  single-use auth keys if you use keys at all. Never commit an auth key.
- Optionally restrict traffic with an [access policy](tailscale.md#optional-restrict-traffic-with-an-access-policy).
- If a password may have leaked, run `./setup.sh --force` on the fusion center,
  restart the stack, and reconfigure every node.
- Keep JetPack, Docker and the base images updated.

## Things to be aware of

- **The dashboard has no login.** Anyone who can open it can start and stop
  nodes and run calibration. That is why it is reachable only from your tailnet.
- **The node service runs as root** (GPIO and radar setup need it), with systemd
  sandboxing.
- Node web pages (visualizer, logs) have no login either.

## Reporting a vulnerability

Please report security problems privately through GitHub's **Report a
vulnerability** button on the affected repository (Security tab), not in a
public issue.

# 1. Your Tailscale network

NetRadar devices talk to each other over [Tailscale](https://tailscale.com), a
private WireGuard network. It works across campus networks, NATs and Wi-Fi
without opening firewall ports, and gives every device a stable name.

Each NetRadar deployment uses **its own** tailnet. Only devices you add can reach
your fusion center and nodes.

## Create the tailnet

1. Sign up at <https://login.tailscale.com> (any supported identity provider).
2. In the admin console, open **DNS** and make sure **MagicDNS** is enabled, so
   devices can be reached by name (e.g. `my-fusion-center`).

## Add each device

On the fusion center and on every Jetson:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

`tailscale up` prints a login link; open it on any computer and approve the
device. No auth key is stored on the device.

Then, in the admin console under **Machines**:

- **Rename** devices to something clear, e.g. `netradar-fc`, `node1`, `node2`.
  Nodes reach the fusion center by this name.
- **Disable key expiry** for the fusion center and the nodes (⋯ menu → *Disable
  key expiry*). Otherwise headless devices drop off the network when their key
  expires.

Check from any device: `tailscale status` lists all your devices, and
`ping netradar-fc` should work.

!!! tip "Unattended installs"
    To add many devices without a browser, create an auth key in the admin
    console and pass it **once**: `sudo tailscale up --auth-key=tskey-...`.
    Prefer short-lived, single-use keys, and never store a key in a file or a
    git repository.

## Optional: restrict traffic with an access policy

By default every device in your tailnet can reach every other. If you share the
tailnet with other machines, you can limit NetRadar traffic with tags and an
access policy (admin console → **Access controls**). A starting point — adapt it
to your tailnet and check the
[Tailscale ACL documentation](https://tailscale.com/kb/1018/acls):

```json
{
  "tagOwners": {
    "tag:netradar-fc":   ["autogroup:admin"],
    "tag:netradar-node": ["autogroup:admin"]
  },
  "acls": [
    // nodes -> fusion center: MQTT broker and NTP
    {"action": "accept", "src": ["tag:netradar-node"], "dst": ["tag:netradar-fc:1883,123"]},
    // people in your tailnet -> dashboard, node web pages, SSH
    {"action": "accept", "src": ["autogroup:member"],
     "dst": ["tag:netradar-fc:5002,22", "tag:netradar-node:5001,5003,22"]}
  ]
}
```

Tag a device when it joins, e.g. `sudo tailscale up --advertise-tags=tag:netradar-node`.

**Next:** [set up the fusion center](fusion-center.md).

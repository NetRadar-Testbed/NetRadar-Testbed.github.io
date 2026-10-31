# Dashboard

The fusion center's dashboard shows every node's detections and tracks live, in
one bird's-eye view, and lets you control the nodes and run calibration. It is
the `dashboard` service of NetRadar-FusionCenter and starts with
`docker compose up`.

Open it from any device on your tailnet:

```text
http://<fusion-center-name>:5002        e.g. http://netradar-fc:5002
```

!!! note
    The page has no login, so it is only reachable on the fusion center's
    Tailscale address (see [Security](security.md)). Your browser also needs
    internet access: the page loads its live-update library (Socket.IO) from
    `cdn.socket.io`.

## The plot

A bird's-eye x–y view, refreshed about 30 times per second:

- **Clusters** — each node's detections (cluster centroids, converted from range
  and angle to x/y), coloured per node.
- **Tracks** — each node's confirmed tracks.
- **Node pos** — the radars' positions, once the network is calibrated.

!!! info "Before calibration the plot is not aligned"
    Until a calibration has run, each node's points are drawn in **its own**
    coordinates, on top of each other. After calibration all nodes are drawn in
    one shared frame, with the first node at the origin, and the radar positions
    appear.

## Sidebar

| Section | What it does |
| :--- | :--- |
| **Nodes** | Every node and whether it is online. |
| **Calibration** | *Group* (default `default`), *Frames* — how many live frames per node to collect (default 150) — and **Start Calibration**. The status line shows progress; when the result arrives, the plot switches to the shared frame. |
| **Node Control** | One row per node with the pipeline state (`ON` + uptime, or `OFF`) and four buttons: **▶** start the pipeline, **■** stop it, **F** initialise the radar and DCA1000 (*FPGA init*), **R** reset them (*FPGA reset*). **Refresh Status** asks all nodes for their status. |
| **MQTT** | The broker host, port and user the dashboard is connected as, and whether it is connected (the password is never shown). |
| **Controls** | Show/hide *Grid*, *Node pos*, *Clusters*, *Tracks*, *Trails* (track history) and *Doppler color* (colour points by radial velocity); *Zoom* in metres per grid division. |
| **Stats** | Refresh rate (FPS), number of points and number of nodes on the plot. |

## Typical session

1. Check every node shows as online under **Nodes**.
2. For each node, press **F** (radar init), then **▶** (start pipeline). Within
   about 10 s the nodes pick up a time-sync tick and points appear.
3. Have one person walk through the area the radars share, set *Frames*, and
   press **Start Calibration** ([details](first-run.md#calibrate)).
4. Watch the fused view; turn on *Trails* to see paths.

To see a single node's raw view (range-Doppler map, CFAR detections), open that
node's own visualizer: `http://<node-name>:5001`.

# Getting started

## What you need

### For each radar node

| Item | Notes |
| :--- | :--- |
| NVIDIA Jetson Orin Nano | JetPack 6.x (Ubuntu 22.04) |
| TI AWR2243BOOST | 77 GHz radar, 3 TX / 4 RX |
| TI DCA1000EVM | Raw ADC capture card, streams to the Jetson over Ethernet |
| USB cable | AWR2243BOOST (FTDI) → Jetson |
| Ethernet cable | DCA1000 → a **dedicated** Jetson Ethernet port |
| Two jumper wires | Trigger: Jetson header pin 8 → AWR2243BOOST J5 pin 9 (SYNC_IN), plus ground (see [Radar nodes](node.md#wiring)) |
| Power supplies | For the Jetson, the radar board and the DCA1000 |

### For the fusion center

Any Linux machine (x86-64 or ARM64) that can run Docker — a desktop, a small
server or a Jetson. It does not need a GPU.

### Accounts

- A free [Tailscale](https://tailscale.com) account. Your devices join **your
  own** private network (a "tailnet").

## Order of steps

1. **[Tailscale network](tailscale.md)** — create it, add the fusion center and nodes.
2. **[Fusion center](fusion-center.md)** — `setup.sh` generates the passwords and
   prints one configuration block per node; `docker compose up` starts the stack.
3. **[Radar nodes](node.md)** — wire the hardware, `sudo ./install.sh`, then
   `./configure.sh` with the node's block.
4. **[Time synchronization](time-sync.md)** — the fusion center becomes the time
   authority; nodes sync to it.
5. **[First run & calibration](first-run.md)** — start the radars from the
   dashboard and calibrate the network.

Rough time: about an hour for the fusion center, and 30–60 minutes per node
(most of it is `install.sh` compiling and installing packages).

## Testing with one node

A single node can run on its own, without a fusion center, using the local
trigger: `sudo ./test/run_tests.sh` in NetRadar-Node. This is a good way to check
the radar hardware before building the network.

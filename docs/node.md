# 3. Radar nodes

Each node is a Jetson Orin Nano running
[NetRadar-Node](https://github.com/NetRadar-Testbed/NetRadar-Node), connected to
an AWR2243BOOST radar and a DCA1000EVM capture card.

<figure markdown="span">
  ![Four NetRadar radar nodes on tripods in a lab, with close-ups of one node's Jetson Orin Nano, DCA1000EVM capture card and AWR2243BOOST radar](assets/img/netradar-testbed.jpg){ width="900" loading=lazy }
  <figcaption>NetRadar nodes. Each node: NVIDIA Jetson Orin Nano (compute + Wi-Fi), TI DCA1000EVM (ADC data capture), TI AWR2243BOOST (mmWave radar).</figcaption>
</figure>

## Wiring

- **AWR2243BOOST + DCA1000EVM:** assemble and power them as described in TI's
  DCA1000EVM user guide.
- **USB:** AWR2243BOOST → Jetson. The radar is configured over this FTDI USB link.
- **Ethernet:** DCA1000 → a **dedicated** Jetson Ethernet port (default name
  `enP8p1s0`). The Jetson uses `192.168.33.30/24` on this port and the DCA1000
  keeps its factory address `192.168.33.180`. Use a different port (or Wi-Fi)
  for the internet and Tailscale.
- **Trigger:** two wires between the Jetson's 40-pin header and the radar board:

    | From (Jetson 40-pin header) | To (AWR2243BOOST) | Purpose |
    | :--- | :--- | :--- |
    | Pin 8 (trigger output) | J5 pin 9 — **SYNC_IN** | Frame trigger |
    | A GND pin (e.g. pin 6) | A GND pin | Common ground |

    The radar is configured for hardware-triggered frames (`triggerSelect=2` in
    `setup_radar/mmwaveconfig.txt`): each **rising edge** on SYNC_IN starts one
    frame. The Jetson drives 3.3 V pulses, which matches the board's default
    3.3 V I/O. The pulse width is 4 µs and the period 50 ms by default
    (`src/hardware_trigger_mqtt/config.h`); the period must stay longer than the
    radar's programmed frame period (20 ms, `periodicity` in
    `mmwaveconfig.txt`).

!!! warning "Do not skip the ground wire"
    Without a common ground between the Jetson and the radar board, the trigger
    pulse is unreliable and frames are missed at random.

## Install

On a freshly flashed Jetson (JetPack 6.x), from your normal user account:

```bash
# Join your tailnet (step 1)
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up

# Get the code and install (20–40 minutes)
git clone https://github.com/NetRadar-Testbed/NetRadar-Node.git
cd NetRadar-Node
sudo ./install.sh
```

If the DCA1000 is on a different port: `sudo ./install.sh --dca-iface <name>`
(`ip -br link` lists the ports).

`install.sh` installs the system packages, the FTDI driver (bundled, checksum
verified), the JETGPIO library, builds the radar and DCA1000 tools and the
trigger programs, creates the Python environment (`.venv`, with the CPU build of
PyTorch), sets up the DCA1000 network connection, and installs the
`netradar-node` service. It is safe to run again.

## Configure

Run `configure.sh` and paste the block the fusion center printed for this node
([previous step](fusion-center.md)), then press Ctrl-D:

```bash
./configure.sh
```

Or pass it as a file or as options:

```bash
./configure.sh --file node1.env
./configure.sh --host netradar-fc --node-id node1 --password <password>
```

It writes `.env` (readable only by you and root, never committed), binds the
node's web pages to its Tailscale address, checks that the broker is reachable,
and restarts the service.

## Time sync and reboot

```bash
sudo ./time-sync/chrony-setup.sh slave <fusion-center-name>
sudo reboot
```

See [Time synchronization](time-sync.md) for checks.

## Verify

```bash
systemctl status netradar-node
journalctl -u netradar-node -f
```

The node appears as **online** on the fusion center dashboard. Its own pages are
at `http://<node-name>:5001` (live visualizer) and `http://<node-name>:5003`
(logs), from devices on your tailnet.

## Testing a node on its own

Without a fusion center, run the interactive test runner and choose the local
trigger:

```bash
sudo ./test/run_tests.sh
```

It can run the live pipeline with the visualizer, capture raw frames, replay a
capture, and export it to MATLAB `.mat`.

## Updating

```bash
git pull
sudo ./install.sh --skip-apt --skip-network
sudo systemctl restart netradar-node
```

**Next:** [time synchronization](time-sync.md).

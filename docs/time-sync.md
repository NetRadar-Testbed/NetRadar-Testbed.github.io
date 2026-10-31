# 4. Time synchronization

Frames from different nodes can only be fused if they were captured at the same
moment. NetRadar does this in two layers:

1. **Clock sync (chrony):** the fusion center is the NTP time authority; every
   node syncs its clock to it, with the internet NTP pool as a fallback.
2. **Trigger alignment (MQTT):** the fusion center's `timesync` service sends a
   `capture/start` tick about every 10 s with an absolute start time. Each node's
   trigger worker fires its GPIO pulses on that shared schedule, using the
   synchronized clock.

The same `chrony-setup.sh` is in both repositories (`time-sync/`).

## Fusion center

```bash
sudo ./time-sync/chrony-setup.sh master
```

It serves the LAN subnet it detects and, when Tailscale is running, the
Tailscale range `100.64.0.0/10`. Use `--allow <CIDR>` to set the subnets
yourself. If a firewall is active it opens NTP (UDP 123).

## Each node

```bash
sudo ./time-sync/chrony-setup.sh slave <fusion-center-name>
```

After about 30 seconds:

```bash
chronyc sources -v    # the fusion center is marked '*' (selected)
chronyc tracking      # "System time" is the current offset
```

On the fusion center, `chronyc clients` lists the nodes that are syncing.

## Measuring sync quality

NetRadar-Node's `time-sync/` folder contains tools to log clock offsets on each
node and compute the worst-case offset between any two devices
(`collect_offsets.py`, `analyze_offsets.py`, `worst_case_offset.py`). See its
[README](https://github.com/NetRadar-Testbed/NetRadar-Node/tree/main/time-sync).

**Next:** [first run and calibration](first-run.md).

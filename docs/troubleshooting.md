# Troubleshooting

## Fusion center

**`docker compose up` says "run ./setup.sh first".**
Run `./setup.sh --nodes <n>` first; the stack has no default passwords.

**"no Tailscale IP found" from `setup.sh`.**
Run `sudo tailscale up` and check `tailscale ip -4`, or pass `--bind-ip <IP>`.

**Dashboard does not load.**
Open it from a device on the same tailnet, using the fusion center's Tailscale
name or IP. If the fusion center's Tailscale IP changed, re-run `./setup.sh`
(it keeps the passwords) and `docker compose up -d`.

**Dashboard loads but nothing updates (no nodes, no points).**
The page needs internet access in your browser to load Socket.IO from
`cdn.socket.io`. Check the browser console for a blocked script. Also check
`docker compose logs dashboard` shows "MQTT connected".

**`docker compose up` fails with "address already in use".**
Another service on the fusion center already uses port 1883 (MQTT), 5002
(dashboard) or 5432 (PostgreSQL). Stop it, or change the left-hand port in
`docker-compose.yml`.

**A node added with `--add-node` cannot connect.**
Restart the broker so it reads the new user: `docker compose restart nanomq`.

## Nodes

**Node is not "online" on the dashboard.**

1. `systemctl status netradar-node` and `journalctl -u netradar-node -f`.
2. Is `.env` present and complete? Re-run `./configure.sh` with the node's block
   (`./setup.sh --print-node <id>` on the fusion center shows it again).
3. Can the node reach the broker? `configure.sh` checks this; manually:
   `timeout 3 bash -c '</dev/tcp/<fusion-center-name>/1883' && echo ok`.
4. Wrong password → the broker rejects the connection; re-run `configure.sh`
   with the current block.

**The service fails with `status=226/NAMESPACE`.**
A path the sandbox needs is missing — usually `.env` (run `./configure.sh`) or
`logs/` (re-run `sudo ./install.sh --skip-apt --skip-network`).

**Radar fails to start / `setup_radar` cannot open the device.**

- Try `sudo ./scripts/reset_radar.sh`, then start again.
- If the FTDI device is claimed by the kernel's serial driver, unload it:
  `sudo rmmod ftdi_sio usbserial`.
- Check the USB cable and that the radar board is powered.

**Radar is configured but no frames arrive, or frames are missed at random.**
The radar only captures when it sees a trigger pulse:

- Check the wiring: Jetson header pin 8 → AWR2243BOOST J5 pin 9 (SYNC_IN), and
  the ground wire between the boards.
- Check the trigger is running: the node status on the dashboard shows the
  trigger state, or use `journalctl -u netradar-node -f`.
- With an oscilloscope, look for 3.3 V, 4 µs pulses every 50 ms on J5 pin 9.
- Some users reported trouble with Rev B boards; TI suggests probing resistor
  R196 to confirm the trigger reaches the radar chip.

**DCA1000 not reachable (no data, `fpga` command fails).**
The DCA1000 does not answer ping. Run `sudo ./scripts/recover_dca_link.sh`, which
resets the link and checks the neighbour table. Check that the dedicated port has
`192.168.33.30/24`: `nmcli connection show netradar-dca1000`.

**Very slow processing / dropped frames.**
Make sure the CPU build of PyTorch is installed
(`.venv/bin/python -c "import torch; print(torch.__version__)"` shows `+cpu` or a
plain version, and `torch.cuda.is_available()` is `False`).

## Time sync

**Node never selects the fusion center (`chronyc sources -v`).**
Check the fusion center is reachable, that its allowed subnets include the node
(`chronyc clients` on the fusion center), and that UDP 123 is open. Large
initial offset: `sudo chronyc makestep`.

## Still stuck?

Open an issue on the relevant repository with the commands you ran and the log
output (remove passwords and IP addresses first).

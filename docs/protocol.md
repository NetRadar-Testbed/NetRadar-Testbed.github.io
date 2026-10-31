# MQTT protocol

All topics start with the prefix `netradar/v1` (`NETRADAR_TOPIC_PREFIX`; it must
be the same on the fusion center and every node). `<gid>` is the group id
(`default`), `<nid>` the node id. Every payload is JSON and carries
`schemaVersion`.

## Topics

### Published by nodes

| Topic | When |
| :--- | :--- |
| `netradar/v1/presence/<nid>` | On connect, periodically, and as last-will (`status: online/offline`, retained) |
| `netradar/v1/group/<gid>/node/<nid>/status` | Every second: pipeline/trigger state, uptime |
| `netradar/v1/group/<gid>/capture/state/<nid>` | State changes (`idle` / `live` / `calib`) |
| `netradar/v1/group/<gid>/capture/ack/<nid>` | Acknowledges every `capture/start` |
| `netradar/v1/group/<gid>/frames/<nid>` | Every live frame (clusters with member detections, tracks) |
| `netradar/v1/group/<gid>/calibration/frame/<nid>` | Calibration-burst frames (burst mode only, see below) |
| `netradar/v1/group/<gid>/calibration/done/<nid>` | After the last calibration-burst frame (burst mode only) |

### Subscribed by nodes

| Topic | Purpose |
| :--- | :--- |
| `netradar/v1/group/<gid>/capture/start` | Time-sync tick or calibration command |
| `netradar/v1/group/<gid>/calibration/result` | Calibration result to apply |
| `netradar/v1/group/<gid>/node/<nid>/command` | Node commands from the dashboard |

### Fusion center

| Topic | Publisher | Purpose |
| :--- | :--- | :--- |
| `netradar/v1/group/<gid>/capture/start` | timesync | Time-sync tick |
| `netradar/v1/server/start/request` | dashboard | Calibration trigger |
| `netradar/v1/server/start/result` | calibration | Calibration armed / rejected |
| `netradar/v1/group/<gid>/calibration/result` | calibration | Calibration result |
| `netradar/v1/server/status` | timesync, calibration | Server online/offline |
| `netradar/v1/group/<gid>/node/<nid>/command` | dashboard | Node commands |

## `capture/start`: time-sync tick vs. calibration

Both use the same topic and message shape (`schemaVersion`, `commandId`,
`startEpochMs`, `captureConfig`). The node looks at `captureConfig.calibration`:

- `false` (or absent) → **time-sync tick.** Take `startEpochMs` as the time
  reference; start live capture aligned to it if not already capturing,
  otherwise re-align; acknowledge with `ready: true` and the `commandId`.
- `true` → **calibration burst.** Align to `startEpochMs`, capture
  `calibrationFrames` frames, publish them to `calibration/frame/<nid>`, then
  `calibration/done/<nid>` with `totalFrames`, and resume live capture. A
  calibration command takes precedence over a time-sync tick.

!!! note "How the current fusion center calibrates"
    The fusion center's calibration service does **not** send calibration
    `capture/start` commands: when you press *Calibrate* it collects the next
    live frames that nodes already stream on `frames/<nid>`. Nodes still support
    the burst mode above for compatibility.

## Node state machine

```
                 ┌─────────────┐
          ┌─────▶│    IDLE     │◀──────────────┐
          │      └──────┬──────┘               │
          │             │ time-sync tick       │
          │      ┌──────▼──────┐               │
          │      │    LIVE     │───────────────┤
          │      │   CAPTURE   │  tick: re-align
          │      └──────┬──────┘               │
          │             │ calibration command  │
          │      ┌──────▼──────┐               │
          │      │ CALIBRATION │               │
          │      │    BURST    ├───────────────┘
          │      └──────┬──────┘  done → back to LIVE
          └─────────────┘
```

- IDLE → LIVE on the first time-sync tick; LIVE → LIVE on each tick (re-align).
- LIVE → CALIBRATION on a calibration-burst `capture/start`; back to LIVE after
  `calibration/done`.
- Any state → IDLE on error or an explicit stop.

## Joining

A node that starts or reconnects publishes `presence` (`online`) and
`capture/state` (`idle`), receives a time-sync tick within about 10 s, and starts
capturing immediately — no calibration is needed first. It applies an identity
calibration until a calibration result arrives, and keeps the most recent result.

## Node commands

`netradar/v1/group/<gid>/node/<nid>/command` with `{"action": ...}`:

| `action` | Effect |
| :--- | :--- |
| `start_radar` | Configure the AWR2243 and DCA1000 and start streaming |
| `reset_radar` | Reset the DCA1000 FPGA and stop the radar setup program |
| `start_pipeline` / `stop_pipeline` | Start/stop the processing pipeline (and trigger) |
| `start_trigger` (`mode`: `mqtt`/`local`) / `stop_trigger` | Control the hardware trigger |
| `status` | Publish status now |

The exact payload fields are defined in the code: `src/node_launcher.py` and
`src/hardware_trigger_mqtt/mqtt_trigger_client.py` (NetRadar-Node), and
`db/init.sql` for what the fusion center stores.

# Architecture

## Roles

| Role | Hardware | Software |
| :--- | :--- | :--- |
| **Radar node** (one per radar) | Jetson Orin Nano + AWR2243BOOST + DCA1000EVM | NetRadar-Node: real-time pipeline, trigger worker, node service |
| **Fusion center** (one per network) | Any Linux machine with Docker | NetRadar-FusionCenter: broker, timesync, calibration, dashboard, PostgreSQL |

All devices share one private Tailscale network. MQTT (topic prefix
`netradar/v1`) carries commands, time-sync ticks, node status and live frames;
see [MQTT protocol](protocol.md).

## Life of a frame

1. **Trigger.** The fusion center's `timesync` service publishes a
   `capture/start` tick with an absolute start time. On every node the trigger
   worker pulses a GPIO line on that shared schedule (default every 50 ms); the
   radar captures one frame per pulse. Chrony keeps the node clocks aligned.
2. **Capture.** The AWR2243 streams raw ADC samples over LVDS to the DCA1000,
   which forwards them to the Jetson as UDP packets.
3. **Process (node).** A multi-process pipeline, each stage pinned to its own
   CPU core:
    - range-Doppler map (windowed 2-D FFT per virtual channel, background
      subtraction);
    - CFAR detection;
    - angle-of-arrival estimation for each detection;
    - 3-D (range, Doppler, angle) DBSCAN clustering and cluster centroids;
    - JPDA multi-target tracking.
4. **Publish.** Each frame's clusters (with their member detections) and tracks
   are published to `netradar/v1/group/<gid>/frames/<nid>`; the node's own
   visualizer shows the range-Doppler map, detections and tracks.
5. **Fuse (fusion center).** The bridge stores every frame in PostgreSQL; the
   dashboard places every node's targets in a shared bird's-eye frame using the
   latest calibration.

## Radar parameters (defaults)

From `src/radar/config.py` and `setup_radar/mmwaveconfig.txt` in NetRadar-Node:

| Parameter | Value |
| :--- | :--- |
| MIMO | 3 TX × 4 RX (TDM), 12 virtual channels |
| Samples per chirp / chirps per TX per frame | 512 / 64 |
| ADC sampling rate / chirp slope | 10 MHz / 83 MHz/µs |
| Range resolution / maximum range | 0.035 m / 18.1 m |
| Velocity resolution / maximum velocity | 0.153 m/s / ±4.9 m/s |
| Frame trigger | Hardware (GPIO), 50 ms period by default |

## Self-calibration

<figure markdown="span">
  ![Diagram: four radars with overlapping fields of view tracking walking people; pairwise self-calibration between radars, then network averaging](assets/img/self-calibration-approach.png){ width="700" loading=lazy }
  <figcaption>Self-calibration: pairs of radars that see the same people moving estimate their relative position and orientation (pairwise self-calibration), and the estimates are combined across the network (network averaging).</figcaption>
</figure>

Calibration runs on the fusion center on demand. It buffers live frames from all
nodes, solves each pair of nodes in closed form with RANSAC to reject outliers,
and averages the pairwise results over direct and indirect paths in the network.
The result gives each node's position and heading in a common frame. The method
is described in the papers on [Citing NetRadar](citing.md).

## More detail

- Node internals, stage by stage:
  [`docs/ARCHITECTURE.md`](https://github.com/NetRadar-Testbed/NetRadar-Node/blob/main/docs/ARCHITECTURE.md)
  in NetRadar-Node.
- Fusion center services:
  [NetRadar-FusionCenter README](https://github.com/NetRadar-Testbed/NetRadar-FusionCenter#services-in-detail).

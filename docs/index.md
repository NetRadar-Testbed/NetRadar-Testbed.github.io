# NetRadar

**NetRadar is an open-source networked mmWave radar testbed.** Several radar
nodes, each a TI AWR2243 radar with an NVIDIA Jetson, capture frames on a shared,
clock-synchronized hardware trigger and stream their detections to a **fusion
center**. The fusion center self-calibrates the network (where every radar is and
which way it faces) and shows all nodes' targets in one bird's-eye view.

This site shows you how to build **your own** NetRadar testbed from scratch, on
your own private network.

<figure markdown="span">
  ![Four NetRadar radar nodes on tripods in a lab, with close-ups of one node's Jetson Orin Nano, DCA1000EVM capture card and AWR2243BOOST radar](assets/img/netradar-testbed.jpg){ width="900" loading=lazy }
  <figcaption>A four-node NetRadar testbed. Each node: NVIDIA Jetson Orin Nano (compute + Wi-Fi), TI DCA1000EVM (ADC data capture), TI AWR2243BOOST (mmWave radar).</figcaption>
</figure>

## See it in action

<figure markdown="span">
  ![Outdoor courtyard with three radars on tripods, labelled Radar 1 in the foreground and Radars 2 and 3 in the distance](assets/img/outdoor-deployment.jpg){ width="800" loading=lazy }
  <figcaption>Outdoor three-radar deployment used in the demo below (at UC Santa Barbara).</figcaption>
</figure>

<figure markdown="span">
  ![Animation: first each radar's tracks in its own coordinates, then the three radars' estimated positions and orientations with the same people aligned across radars](assets/img/self-calibration-demo.gif){ width="800" loading=lazy }
  <figcaption>Self-calibration in action: first each radar's tracks in its own coordinates, then the estimated radar positions and orientations (Radar 1 as reference) with the same people aligned across all three radars.</figcaption>
</figure>

## How it fits together

```
   Radar node 1          Radar node 2          Radar node N
 ┌──────────────┐     ┌──────────────┐      ┌──────────────┐
 │ AWR2243 +    │     │ AWR2243 +    │      │ AWR2243 +    │
 │ DCA1000 +    │     │ DCA1000 +    │ ...  │ DCA1000 +    │
 │ Jetson       │     │ Jetson       │      │ Jetson       │
 └──────┬───────┘     └──────┬───────┘      └──────┬───────┘
        │   detections, status (MQTT over your Tailscale network)
        └────────────┬───────┴─────────────────────┘
              ┌──────▼──────────────────────────────┐
              │ Fusion center (Docker)              │
              │ broker · time-sync ticks ·          │
              │ self-calibration · dashboard · DB   │
              └─────────────────────────────────────┘
```

## The pieces

| Repository | What it is |
| :--- | :--- |
| [NetRadar-Node](https://github.com/NetRadar-Testbed/NetRadar-Node) | Software for each radar node: real-time radar pipeline, trigger, installer |
| [NetRadar-FusionCenter](https://github.com/NetRadar-Testbed/NetRadar-FusionCenter) | Server stack: MQTT broker, time sync, calibration, dashboard, database |

## Where to start

1. [Getting started](getting-started.md) — what you need and the order of steps.
2. [Your Tailscale network](tailscale.md) — the private network your devices share.
3. [Fusion center](fusion-center.md), then [radar nodes](node.md).
4. [Time synchronization](time-sync.md) and [first run & calibration](first-run.md).

!!! note "Your network, your credentials"
    Nothing in NetRadar points at an existing deployment. You create your own
    Tailscale network, and the setup scripts generate fresh random passwords on
    your machines.

## Citing

If you publish work that uses NetRadar or a testbed built with it, please
[cite our papers](citing.md) and state that your testbed was built with NetRadar.

NetRadar was developed as part of the **Chirp Capstone Project 2025–2026,
University of California, Santa Barbara** — see [About](about.md).

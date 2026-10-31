# NetRadar

**NetRadar is an open-source networked mmWave radar testbed.** Several radar
nodes, each a TI AWR2243 radar with an NVIDIA Jetson, capture frames on a shared,
clock-synchronized hardware trigger and send what they see to a **fusion
center**. The fusion center works out where every radar is and which way it
faces, then shows all of their targets together in one bird's-eye view.

This site walks you through building **your own** NetRadar testbed, from an empty
network to a calibrated set of radars.

<figure markdown="span">
  ![Four NetRadar radar nodes on tripods in a lab, next to close-ups of one node's Jetson Orin Nano, DCA1000EVM capture card and AWR2243BOOST radar](assets/img/netradar-testbed.jpg){ width="900" loading=lazy }
  <figcaption>Our four-node testbed. Each node pairs an NVIDIA Jetson Orin Nano with a TI DCA1000EVM capture card and a TI AWR2243BOOST radar.</figcaption>
</figure>

## See it in action

<div class="grid" markdown>

<figure markdown="span">
  ![Three radars on tripods in a courtyard, labelled Radar 1 in front and Radars 2 and 3 in the distance](assets/img/outdoor-deployment.jpg){ loading=lazy }
  <figcaption>The setup: three radars in a courtyard at UC Santa Barbara.</figcaption>
</figure>

<figure markdown="span">
  ![Animation: each radar's tracks in its own coordinates, then the radars' estimated positions and orientations with the tracks lined up](assets/img/self-calibration-demo.gif){ loading=lazy }
  <figcaption>Each radar starts out seeing people in its own coordinates. Calibration finds where every radar is and which way it faces, and the tracks line up.</figcaption>
</figure>

</div>

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

!!! tip "Your own testbed"
    Everything runs on your own Tailscale network, and the setup scripts create
    fresh passwords for each testbed, so you're in control from the start.

## Citing

**Using NetRadar in your research?** Please [cite our papers](citing.md), and
mention in your paper that your testbed was built with NetRadar.

## Contributing

Contributions are welcome! Please fork the repository, make your changes on a branch
and open a pull request against `main`. More details in
[CONTRIBUTING.md](https://github.com/NetRadar-Testbed/.github/blob/main/CONTRIBUTING.md).

NetRadar was developed as part of the **Chirp Capstone Project 2025–2026 at the
University of California, Santa Barbara**. Meet the team on the [About](about.md)
page.

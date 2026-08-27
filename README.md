# closed-loop-nwdaf

**A standards-compliant NWDAF for 5G core networks, and the closed loops it drives.**

[![CI](https://github.com/fatemeshafiee/oai-cn5g-nwdaf/actions/workflows/ci.yml/badge.svg)](https://github.com/fatemeshafiee/oai-cn5g-nwdaf/actions/workflows/ci.yml)
[![Paper (NOMS 2026)](https://img.shields.io/badge/paper-NOMS%202026-blue)](https://arxiv.org/abs/2505.06789)
[![Paper (ITU J-FET)](https://img.shields.io/badge/paper-ITU%20J--FET%207(2)-blue)](https://www.itu.int/pub/S-JNL-VOL7.ISSUE2-2026-A12)

This project implements the 5G Network Data Analytics Function (NWDAF) end to end and shows how it can
close the loop on network operations without human intervention: collect telemetry from live user traffic,
analyse it with ML models, and act on the result automatically.

It includes the first open-source implementation of the **UPF Event Exposure Service** (3GPP TS 29.564) reporting usage, QoS and volume measurements at
per-flow and per-session granularity, an **ML model provisioning service** built on MLflow, and
extensions to the **SMF and PCF** that consume NWDAF analytics to mitigate problems at runtime.

## How the loop works

```
  ┌──────────────────┐  per-flow and per-session  ┌──────────────────┐  models   ┌────────────────────┐
  │  UPF + EES (C)   │ ─────────────────────────► │      NWDAF       │ ◄──────── │  MLModelProvision  │
  │  open5gs-ees     │  usage, QoS, volume        │  oai-cn5g-nwdaf  │           │  (MLflow registry) │
  └──────────────────┘                            └────────┬─────────┘           └────────────────────┘
            ▲                                              │ analytics
            │                                              ▼
            │                                     ┌──────────────────┐
            │                                     │  SMF  ·  PCF *   │
            │                                     │ policy and action│
            │                                     └────────┬─────────┘
            │      session release, rate limiting          │
            └──────────────────────────────────────────────┘
```

\* The SMF loop (bot detection, PDU session release) is in `open5gs-ees`. The UPF QoS monitoring
reports, the final congestion-estimation engine, and the PCF loop (rate adaptation under congestion,
from the ITU J-FET paper) are implemented and evaluated but not yet released publicly.

## Results

Measured on a Kubernetes testbed with UERANSIM.

| Property | Result |
|---|---|
| Telemetry overhead on the forwarding path | under **0.11 ms** added one-way delay, under **6.5% CPU** at 1 Gbps |
| Scaling with concurrent users | 50 to 200 UEs at fixed 200 Mbps costs **+36 millicores**, **+2.8 MiB** |
| Bot detection, attack to mitigation | **under one second**, detection latency reduced **33%** by tuning the reporting interval |
| Congestion control on guaranteed traffic | packet loss **8-10% to about 1%**, RTT **12-15 ms to about 3 ms** |
| Guaranteed bitrate under contention | voice flow restored to its **64 kbps** guarantee |

## Repositories

This work spans four repositories, each covering one part of the system:

- **UPF Event Exposure Service** — [`open5gs-ees`](https://github.com/fatemeshafiee/open5gs-ees)
  Modified Open5GS core in C, including the UPF Event Exposure Service (3GPP TS 29.564) that streams
  per-flow and per-session telemetry off the packet-forwarding path, and the SMF extensions that release malicious
  PDU sessions when the loop closes.

- **NWDAF analytics and engines** — [`oai-cn5g-nwdaf`](https://github.com/fatemeshafiee/oai-cn5g-nwdaf)
  Extended OAI-NWDAF: the UPF-EES client, the bot-detection engine (graph-based features on live
  telemetry) with its analytics and subscription handling, and the integration with model
  provisioning. The congestion component here contains the data collection and model-serving client;
  the final estimation logic is not yet released.

- **ML model provisioning** — [`MLModelProvision`](https://github.com/fatemeshafiee/MLModelProvision)
  MLflow-based service managing the model lifecycle (artifacts, versions, selection) and serving models
  to NWDAF components at runtime, behind a custom management API.

- **Deployment and testbed** — [`open5gs-k8s-nwdaf`](https://github.com/fatemeshafiee/open5gs-k8s-nwdaf)
  Kubernetes deployment of the whole stack, extended from
  [`open5gs-k8s`](https://github.com/niloysh/open5gs-k8s) to add the NWDAF and MLflow services.
  **Start here** to run the system end to end.

> **Release status.** Public here: the UPF Event Exposure Service (usage and volume measurements),
> the NWDAF **bot-detection** engine and its analytics, the model provisioning service, and the
> SMF-based mitigation loop. Implemented and evaluated in the papers but not yet released: the UPF
> QoS monitoring reports, the final **congestion-estimation** engine, and the PCF-based rate
> adaptation from the ITU J-FET paper.

## Getting started

Deploy the full stack from [`open5gs-k8s-nwdaf`](https://github.com/fatemeshafiee/open5gs-k8s-nwdaf),
which brings up the Open5GS core with the EES-enabled UPF, the NWDAF components, the MLflow model
provisioning service, and a UERANSIM RAN for traffic generation.

## Papers

If you use this code or build on it, please cite:

> F. Shafiei Ardestani, N. Saha, N. Limam, and R. Boutaba,
> "Towards NWDAF-enabled Analytics and Closed-Loop Automation in 5G Networks",
> IEEE/IFIP Network Operations and Management Symposium (NOMS), 2026.
> [arXiv:2505.06789](https://arxiv.org/abs/2505.06789)

> F. Shafiei Ardestani, N. Limam, and R. Boutaba,
> "Towards zero-touch network orchestration: An NWDAF-centered standards-compliant closed-loop approach",
> ITU Journal on Future and Evolving Technologies, Vol. 7, Issue 2, 2026.
> [Publication](https://www.itu.int/pub/S-JNL-VOL7.ISSUE2-2026-A12)

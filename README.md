# closed-loop-nwdaf

This project implements a practical, standards-compliant realization of the 5G Network Data Analytics Function (NWDAF) and demonstrates how it can drive closed-loop security automation in modern 5G core networks.

The system introduces the first implementation of the UPF Event Exposure Service (EES) for standardized, real-time traffic monitoring; an ML model provisioning service built on MLflow for managing analytics models; and extensions to the Session Management Function (SMF) that consume NWDAF "abnormal UE behavior" analytics to automatically mitigate attacks by releasing malicious PDU sessions.

Ecsvaluation shows that the closed-loop pipeline can detect and react to threats with low latency, making NWDAF a practical enabler of zero-touch security management in 5G networks.

## Repository overview

This work is implemented across multiple repositories, each covering a specific part of the closed-loop NWDAF system:

- **Open5GS (Core Source Code)** – [`open5gs-ees`](https://github.com/fatemeshafiee/open5gs-ees.git)  
  Contains the modified Open5GS core, including the implementation of the UPF Event Exposure Service (EES) and the SMF extensions used to trigger closed-loop mitigation actions (releasing malicious PDU sessions).

- **Kubernetes Deployment & Testbed** – [`open5gs-k8s-nwdaf`](https://github.com/fatemeshafiee/open5gs-k8s-nwdaf.git)  
  This repository is based on the original [`open5gs-k8s`](https://github.com/niloysh/open5gs-k8s.git) which provides a Kubernetes-based deployment of the Open5GS 5G core and UERANSIM RAN and UEs. In this work, we extend it to integrate the NWDAF and the MLflow-based Model Provisioning Service. This repository is the main entry point for deploying the end-to-end testbed used in the paper’s evaluation.

- **ML Model Provisioning Service** – [`MLModelProvision`](https://github.com/fatemeshafiee/MLModelProvision.git)  
  Implements the ML Model Provisioning Service, built on MLflow, which manages the lifecycle of analytics models (training artifacts, versions, and selection) and serves models to the NWDAF components at runtime.

- **NWDAF Analytics & Bot Detection** – [`oai-cn5g-nwdaf`](https://github.com/fatemeshafiee/oai-cn5g-nwdaf.git)  
  Contains the extended OAI-NWDAF implementation, including the UPF-EES client, our bot-detection analytics engine, and the integration with the ML Model Provisioning Service to retrieve ML models and compute “abnormal UE behavior” analytics.
  
These repositories provide the reference implementation and artifacts for the following paper. If you use this code or build on it, please cite:

Fatemeh Shafiei Ardestani, Niloy Saha, Noura Limam, and Raouf Boutaba,
“Towards NWDAF-enabled Analytics and Closed-Loop Automation in 5G Networks”,
arXiv:2505.06789, 2025.

# Project 002: Secure Local LLM Deployment

## Overview

This project documents the deployment and security validation of **Lupin**, a locally hosted large language model running on an Ubuntu Server virtual machine within an isolated Proxmox homelab. The objective was to establish a functional, locally operated AI service while examining the practical constraints of limited hardware and validating relevant network and service security controls.

The deployment uses **Ollama** and a **Qwen3 4B** model in **Q4_K_M** quantised GGUF format. It operates without a dedicated GPU and does not require an external AI service for inference.

## Environment

| Component | Configuration |
| --- | --- |
| Hypervisor | Proxmox VE on HP EliteDesk (Alfred) |
| Host hardware | Intel Core i5-7500T; 16 GB installed RAM |
| Guest | Ubuntu Server 24.04 LTS, VM 100 |
| Guest resources | 6 GiB RAM; CPU allocation increased from 2 to 3 virtual cores |
| Guest storage | LVM-backed root filesystem expanded from 15 GiB to 30 GiB |
| LLM runtime | Ollama, managed by systemd |
| Model | Qwen3 4B, Q4_K_M quantisation, registered as `lupin:latest` |
| Lab gateway | OPNsense (`10.20.0.1`) |

## Work Completed

- Assessed host and guest resources and expanded the Ubuntu root logical volume using available LVM capacity.
- Increased VM memory and compared model performance at different virtual CPU allocations.
- Installed Ollama using an archive transferred over SSH from a laboratory machine with controlled internet access, validating the archive's transfer integrity using SHA-256.
- Registered a locally stored GGUF model through an Ollama Modelfile.
- Configured a systemd service with a dedicated unprivileged account, filesystem restrictions, automatic startup and restart-on-failure behaviour.
- Restricted the Ollama API listener to `127.0.0.1:11434` and tested connectivity from another laboratory machine.
- Tested selected outbound network paths and verified an OPNsense firewall denial for an attempted public HTTPS connection.

## Validation and Results

**Functional deployment:** Qwen3 4B operated locally on CPU-only infrastructure, although inference was relatively slow.

**Network controls:** An outbound HTTPS connection from Ubuntu to a public IP address was denied by OPNsense. A TCP connection from Kali (`10.20.0.121`) to Ubuntu's laboratory address (`10.20.0.129`) on port `11434` was refused, consistent with Ollama's loopback-only listener. These tests support the documented controls for the specific paths examined, not an exhaustive guarantee of isolation.

**Performance:** Increasing the VM allocation from two to three virtual CPU cores increased observed generation speed from **1.61 to 2.45 tokens per second** (approximately **52%**) in the recorded comparison. Differences in generated token counts also affected total response time.

## Documentation

- [Technical deployment and security assessment](Report/secure-local-llm-deployment.md)
- Supporting screenshots are stored in [`Evidence/`](Evidence/).

## Limitations and Future Work

The laboratory host has limited CPU and memory capacity, and model response times were substantial. Service-hardening configuration was inspected, but individual restrictions were not exhaustively penetration-tested. Future work may explore resource optimisation or additional functionality following a separate security review.

This project was completed in a controlled, non-production homelab environment. Findings are limited to the systems, configurations and tests documented in the technical report.

# Project 002: Secure Local LLM Deployment

**Analyst:** Orlo
**Date:** 8 October 2026
**Status:** Final technical draft (evidence review pending)

## 1. Executive Assessment

Project 002 evaluated the deployment of a locally hosted large language model within an isolated Proxmox virtual machine, with particular emphasis on Linux administration, controlled network access and the practical limitations of CPU-only inference. A quantised Qwen3 4B model, registered as `lupin`, was deployed through Ollama on Ubuntu Server (VM 100). The project included virtual machine resource adjustments, logical volume expansion, service configuration and security testing.

The deployment achieved functional local inference, and the tests performed supported the intended network restrictions. OPNsense logs confirmed denial of a tested outbound HTTPS connection, while a connection attempt from Kali Linux to Ubuntu's address on TCP port 11434 was refused, consistent with Ollama listening only on the loopback interface. The Ollama service was configured to run under a dedicated unprivileged account with additional systemd hardening settings; these controls were reviewed but not subjected to a full adversarial assessment.

Performance was the principal limitation. Increasing the Ubuntu VM's virtual CPU allocation from two to three cores improved measured generation speed from 1.61 to 2.45 tokens per second, although response times remained substantial. The project established a usable, security-conscious local deployment baseline and identified the hardware trade-offs involved in running language models without dedicated GPU acceleration.

## 2. Objective

The objective of this project is to deploy and secure a locally hosted large language model (LLM) within an isolated laboratory environment.

The deployment will focus on network isolation, controlled access, system hardening, and validation of security controls to ensure the LLM operates without unauthorised connectivity to the internet or trusted network.

The project will also assess the hardware requirements and practical limitations of running an LLM on locally available infrastructure.

### 2.1 Intended System Functionality

The locally hosted LLM, designated Lupin, will serve as a cybersecurity education and technical assistance system within the isolated laboratory environment.

Its primary functions will include:

- Explaining cybersecurity concepts, methodologies, and tools.
- Supporting Linux administration and network troubleshooting.
- Assisting with Bash and Python scripting.
- Developing scripts for laboratory automation.
- Supporting security testing, defensive analysis, and technical documentation.

The initial deployment will operate in an advisory capacity only.

The LLM will not be granted direct access to execute commands, modify infrastructure, or interact autonomously with laboratory systems.

Future development may introduce controlled automation capabilities, subject to additional security assessment and access restrictions.

The system will operate without internet connectivity, with its functionality and performance evaluated against the available hardware resources.

## 3. Scope

### 3.1 In Scope

The project covers the deployment and security assessment of a locally hosted large language model (LLM), designated Lupin, within the isolated laboratory environment.

The assessment includes:

- Evaluation of available hardware resources and deployment requirements.
- Installation and configuration of the LLM runtime and selected model.
- Network isolation from the internet and trusted network.
- Operating system and service hardening.
- Validation of network restrictions and security controls.
- Basic functional and performance testing.

Lupin will initially function as a cybersecurity education and technical assistance system, supporting conceptual learning, scripting, troubleshooting, and laboratory documentation.

### 3.2 Out of Scope

The following activities are excluded from the initial deployment:

- Internet-connected AI services or external model APIs.
- Autonomous command execution or infrastructure modification.
- Direct access to other laboratory systems by the LLM.
- Production deployment or access by external users.
- Advanced model fine-tuning or training.
- Retrieval-augmented generation (RAG), which will be addressed in a subsequent project.

Future development may introduce controlled automation capabilities following additional security assessment.

## 4. Environment

### 4.1 Existing Hardware Allocation

The target Ubuntu Server virtual machine (VM 100) is hosted
on Proxmox VE.

At the time of assessment, the VM was allocated:

- Memory: 2 GB
- CPU: 2 cores

The Proxmox host (alfred) has 16 GB of installed RAM,
with approximately 15.5 GiB reported as total usable
physical memory.

The existing Ubuntu Server VM allocation of 2 GB
is likely insufficient for the intended LLM workload.

### 4.2 Host Resource Utilisation

With all four laboratory virtual machines running under relatively idle conditions, the Proxmox host reported 6.94 GiB of memory utilisation from a total of 15.50 GiB (44.77%).

Approximately 8.56 GiB remained unused at the time of measurement. 

This represents an idle baseline rather than a peak-load assessment.

### 4.3 Resource Allocation Considerations

The Proxmox host is dedicated exclusively to laboratory activities and does not support production workloads.

This allows virtual machine resources to be adjusted according to experimental requirements. However, sufficient capacity must be retained for the hypervisor and supporting laboratory systems.

The initial assessment indicates that additional memory could be allocated to the Ubuntu Server VM. The final allocation will depend on the requirements of the selected LLM and the resources needed by other virtual machines.

### 4.4 Graphics Hardware Assessment

A PCI hardware inspection of the Proxmox host identified an Intel HD Graphics 630 integrated graphics controller.

No dedicated GPU was identified in the PCI device listing.

The initial LLM deployment will therefore be designed around CPU-based inference, without relying on dedicated GPU acceleration.

This introduces potential performance limitations, particularly regarding model size, inference speed, and memory consumption.

The suitability of the available hardware will be evaluated during model selection and performance testing.

### 4.5 Hardware Upgrade Strategy

The laboratory operates on an HP EliteDesk desktop computer repurposed as a Proxmox virtualization host.

The initial deployment will use existing hardware to establish a functional, security-controlled LLM environment and evaluate practical performance limitations.

A possible upgrade to 32 GB of system memory and additional storage was considered, subject to hardware compatibility and cost, but was not required for this deployment.

Longer-term expansion may involve a dedicated AI workstation with GPU acceleration if testing demonstrates that the existing platform cannot adequately support the intended workloads.

The deployment will prioritise security validation and operational experience over model size or inference performance.

### 4.6 Processor Assessment

The Proxmox host is equipped with an Intel Core i5-7500T processor operating at a base frequency of 2.70 GHz.

The processor provides four physical cores and four threads. Combined with the absence of a dedicated GPU, this limits
the host's suitability for computationally intensive AI workloads.

The initial deployment will therefore prioritise a small, quantised language model suitable for CPU-based inference.

Model performance will be evaluated during deployment to establish realistic operating limitations.

### 4.7 Virtual Machine Storage Assessment

The Ubuntu Server virtual machine (VM 100) has a 32 GiB virtual disk allocated.

This provides a starting point for installing an LLM runtime and a small, quantised language model.

Available disk space within the guest operating system has not yet been verified.

Storage requirements will be reassessed following model selection and deployment.

A filesystem inspection using `df -h /` identified a 15 GiB root filesystem, with 4.7 GiB used and 9.3 GiB available.

Although the virtual disk is allocated 32 GiB, the root filesystem currently exposes only 15 GiB.

The system uses an LVM-backed root filesystem. Further investigation is required to determine whether additional disk capacity can be allocated without expanding the underlying virtual disk.

An inspection of the Logical Volume Manager (LVM) configuration using `sudo vgs` identified approximately 15 GiB of unallocated capacity within the `ubuntu-vg` volume group.

This capacity can potentially be assigned to the existing root logical volume without increasing the virtual disk allocation in Proxmox. The root filesystem will be expanded to make this capacity available for the LLM deployment.

## 5. Methodology

The deployment and security assessment followed a structured approach comprising six phases.

### 5.1 Baseline Assessment

Document the existing virtual machine configuration, host resources, operating system, and network connectivity.

### 5.2 Model Selection

Evaluate locally deployable language models against available CPU, memory, storage, and intended functionality.

### 5.3 Deployment

Configure the Ubuntu Server virtual machine, install the selected LLM runtime, and deploy the model.

### 5.4 System Hardening

Apply appropriate operating system, service, and network security controls to restrict unauthorised access and connectivity.

### 5.5 Security Validation

Test network isolation, service accessibility, and relevant security controls to verify that the deployment meets its defined requirements.

### 5.6 Functional and Performance Testing

Evaluate the LLM's ability to perform its intended educational and technical assistance functions while monitoring resource utilisation and operational limitations.

## 6. Findings

The available Proxmox infrastructure supported a functioning quantised Qwen3 4B model, but required adjustments to the Ubuntu VM's allocated resources. Expanding the LVM-backed root filesystem from 15 GiB to 30 GiB and increasing guest memory from 2 GiB to 6 GiB provided sufficient capacity for the tested deployment. The physical host's four-core processor and lack of dedicated GPU acceleration remained significant constraints on inference performance.

The security assessment found that outbound internet access was restricted for the tested traffic and that Ollama's API was not reachable from the tested Kali host over Ubuntu's laboratory IP address. A matching OPNsense firewall log confirmed denial of the outbound HTTPS attempt. Ollama ran under a dedicated unprivileged account, with systemd configuration settings intended to restrict privileges, filesystem writes and remote API exposure. Configuration inspection and connectivity tests support these conclusions within the scope tested; they do not constitute proof against every possible access path or attack.

Initial performance tests showed that disabling model reasoning reduced the number of generated tokens and therefore response time, without materially changing token generation speed. Increasing the VM allocation to three virtual CPU cores raised generation speed by approximately 52% in the measured comparison. The deployment is functional, but its speed limits suitability for sustained interactive use on the present hardware.

## 7. Implementation

### 7.1 Root Filesystem Expansion

Prior to modifying the storage configuration, a Proxmox snapshot of Ubuntu Server (VM 100) was taken to provide a rollback point.

The existing root logical volume, `ubuntu-lv`, was expanded using previously unallocated capacity within the `ubuntu-vg` volume group.

The following command was executed:

`sudo lvextend -l +100%FREE -r /dev/ubuntu-vg/ubuntu-lv`

The operation increased the logical volume from 15 GiB to 30 GiB. The `-r` option also resized the filesystem online, without requiring a reboot.

The command completed successfully, with no errors reported.

### 7.2 Virtual Machine Memory Expansion

Following the initial resource assessment, Ubuntu Server (VM 100) was shut down to allow its memory allocation to be increased.

Using the Proxmox hardware configuration interface, the VM's allocated memory was increased from 2 GiB to 6 GiB. Memory ballooning was disabled to provide a predictable allocation for LLM inference.

The virtual machine was subsequently restarted to apply the updated configuration.

### 7.3 Offline Software Transfer

The Ollama Linux AMD64 installation archive was downloaded from the official Ollama website onto Kali Linux, which has permitted internet access.

The archive was tested using `zstd -t` to check its compressed-data integrity. A SHA-256 hash was then calculated using `sha256sum`.

The archive was transferred from Kali Linux to Ubuntu Server using SCP over an SSH connection. Ubuntu's SSH host key fingerprint had previously been independently verified against the fingerprint obtained through the Proxmox console.

Following the transfer, the SHA-256 hash was recalculated on Ubuntu and confirmed to match the original hash.

This verified that the installation archive was transferred without modification, while Ubuntu remained isolated from the internet.

### 7.4 Ollama Runtime Installation

The Ollama Linux AMD64 archive was inspected to identify its executable and supporting library structure.

The verified archive was extracted into `/usr/local` using the following command:

`sudo tar --zstd -xf ~/ollama-linux-amd64.tar.zst -C /usr/local`

The installation was checked using `/usr/local/bin/ollama --version`, which reported Ollama client version 0.40.1.

The command also reported that no Ollama server instance was running. This was expected because the service had not yet been configured or started.

The runtime was installed without granting Ubuntu direct internet access.

### 7.5 Ollama Service Configuration and Hardening

Ollama was configured as a systemd service using a custom unit file located at `/etc/systemd/system/ollama.service`.

The service was enabled to start automatically at boot and configured to execute under a dedicated, unprivileged `ollama` user and group, following the principle of least privilege.

The service configuration included additional systemd security restrictions:

- `NoNewPrivileges=true` prevents the service and its child processes from acquiring additional privileges through mechanisms such as setuid executables.
- `ProtectSystem=strict` restricts write access to most of the system filesystem.
- `ProtectHome=true` restricts access to user home directories.
- `ReadWritePaths=/usr/share/ollama` permits write access to the designated Ollama data directory.
- `PrivateTmp=true` provides the service with isolated temporary directories.

The `OLLAMA_HOST` environment variable was configured as `127.0.0.1:11434`, restricting the Ollama API listener to the local loopback interface rather than exposing it to other systems on the laboratory network.

The service was also configured with `Restart=on-failure` to support automatic recovery following unexpected process failures.

### 7.6 Language Model Deployment

The Qwen3 4B language model was deployed locally using the Ollama runtime on Ubuntu Server (VM 100). The model was selected for evaluation on the laboratory's CPU-based infrastructure, which lacks a dedicated GPU and has limited memory and processing resources.

The model was stored in GGUF format at `/usr/share/ollama/Qwen3-4B-Q4_K_M.gguf`. A Modelfile was created at `/usr/share/ollama/Modelfile`, containing a `FROM` instruction referencing the local GGUF file. The model was subsequently registered with Ollama under the name `lupin`.

Inspection using `ollama show lupin` confirmed the Qwen3 architecture, approximately four billion parameters and Q4_K_M quantisation. The registered model occupies approximately 2.5 GB of storage. Quantisation reduces the memory required to represent the model's weights, making local inference more practical on hardware without dedicated GPU acceleration.

The model supports text generation, reasoning and tool-use capabilities. Reasoning was temporarily disabled during selected performance tests to assess its effect on response time and token generation speed, without permanently removing that capability.

The deployment operates locally without requiring external AI services during inference. Performance results, including comparisons between reasoning settings and virtual CPU allocations, are documented in Sections 8.7 and 8.8.


## 8. Validation

### 8.1 Root Filesystem Expansion Validation

Following the logical volume expansion, the `df -h /` command was executed to verify the resulting filesystem capacity.

The command confirmed that the root filesystem had increased from 15 GiB to 30 GiB, with approximately 24 GiB available and utilisation reduced from 34% to 17%.

The results demonstrate that the additional storage capacity was successfully allocated and made available to the operating system without requiring a reboot.

### 8.2 Memory Expansion Validation

Following the memory allocation change, the `free -h` command was executed within Ubuntu Server.

The operating system reported 5.8 GiB of total memory, with approximately 5.4 GiB available under idle conditions.

These results confirm that the revised memory allocation was successfully recognised by the guest operating system.

The allocation provided additional capacity for the deployed model; inference performance was subsequently evaluated in Sections 8.7 and 8.8.

### 8.3 Initial Network Isolation Validation

The Ubuntu Server virtual machine was identified as `10.20.0.129`, with its default gateway configured as `10.20.0.1` (OPNsense).

An initial connectivity test was performed using `ping -c 4 1.1.1.1` to assess outbound ICMP connectivity to a public IP address.

The test returned 100% packet loss, with four packets transmitted and no responses received.

This result is consistent with the outbound network restrictions implemented during Project 001. However, additional protocol testing is required before concluding that internet connectivity is fully restricted.

An additional test was performed using `curl -I --connect-timeout 5 https://example.com` to assess outbound HTTPS connectivity.

The request failed with a DNS resolution timeout after approximately five seconds.

This result demonstrates that the hostname could not be resolved during the test. However, it does not independently establish whether outbound HTTPS connections to public IP addresses are blocked.

A further HTTPS connectivity test was performed using `curl` with the `--resolve` option to supply a public IP address directly, bypassing DNS resolution.

The connection attempt to TCP port 443 timed out after approximately five seconds.

Combined with the unsuccessful ICMP and DNS tests, this result is consistent with the intended outbound network restrictions. Firewall log inspection is required to confirm that OPNsense is responsible for blocking the traffic.

OPNsense firewall logs were inspected to determine whether the unsuccessful connectivity tests resulted from firewall enforcement.

A matching log entry was identified showing TCP traffic originating from Ubuntu Server (`10.20.0.129`) and destined for the public IP address `93.184.215.14` on port 443.

The connection was blocked on the LAN interface under the `Default deny / state violation` rule.

This confirms that OPNsense denied the tested outbound HTTPS connection, providing direct evidence that the network isolation policy is being enforced for this traffic.

### 8.4 Internal Network Connectivity Validation

Connectivity between Kali Linux (`10.20.0.121`) and Ubuntu Server (`10.20.0.129`) was tested using `ping -c 4 10.20.0.129`.

All four ICMP requests received responses, resulting in 0% packet loss and an average round-trip time of 0.448 ms.

This confirms basic network connectivity between the two laboratory systems. SSH connectivity and host identity were subsequently validated, as documented in Section 8.5.

### 8.5 SSH Connectivity and Host Identity Validation

An SSH connection was established from Kali Linux to Ubuntu Server (`10.20.0.129`).

Before completing authentication, Ubuntu's ED25519 host key fingerprint was independently retrieved through the Proxmox console and compared against the fingerprint presented to Kali.

The fingerprints matched, confirming consistency between the SSH endpoint and the independently inspected host key.

Following successful password authentication, an interactive SSH session was established using the `orlo` account.

This confirms that administrative access to Ubuntu is available from Kali over the isolated laboratory network.

### 8.6 Ollama Service Isolation Validation

Following deployment, the Ollama service was confirmed to be active and running under the dedicated, unprivileged `ollama` account.

The command `sudo ss -ltnp | grep 11434` confirmed that the service was listening exclusively on `127.0.0.1:11434`.

This verifies that the Ollama API is bound to the Ubuntu Server loopback interface rather than its externally accessible network interface.

Remote accessibility was subsequently tested from Kali Linux, as documented in Section 8.9.

### 8.7 Initial LLM Performance Assessment

The Qwen3 4B model was benchmarked on Ubuntu Server with two virtual CPU cores and 6 GiB of allocated memory.

With reasoning mode enabled, a basic firewall explanation required approximately 237 seconds, generating 342 tokens at 1.49 tokens per second.

Disabling reasoning mode reduced the observed response time to approximately 99 seconds, generating 73 tokens at 1.61 tokens per second.

The reduction in response time was primarily attributable to fewer generated tokens rather than a substantial increase in inference speed.

These preliminary results demonstrate that the model operates successfully on the available hardware but exhibits significant performance limitations.

### 8.8 CPU Allocation Performance Comparison

Following the initial performance assessment, Ubuntu Server's CPU allocation was increased from two to three virtual cores.

The Qwen3 4B model was benchmarked using the same firewall explanation prompt, with reasoning mode disabled.

Generation speed increased from 1.61 to 2.45 tokens per second, representing an improvement of approximately 52%.

Total response time decreased from approximately 99 seconds to 33 seconds. This reduction was influenced by both improved processing performance and a lower number of generated tokens.

The results indicate that additional CPU allocation can improve inference performance on the available hardware. Further testing would be required to determine the optimal allocation under concurrent laboratory workloads.

### 8.9 Remote API Accessibility Validation

A remote TCP connectivity test was conducted from Kali Linux (`10.20.0.121`) against Ubuntu Server (`10.20.0.129`) on port 11434 using `nc -nvz -w 3 10.20.0.129 11434`. The attempt returned `Connection refused`, confirming that a TCP connection to Ollama's API could not be established from that host through Ubuntu's laboratory network interface.

Together with the local listener inspection in Section 8.6, this result supports the conclusion that the Ollama service was accessible only through the loopback interface during testing. This validation was limited to the tested network path and port; it did not examine every possible route to the VM. The test is recorded in `evidence-25-ollama-remote-access-denied.png`.

## 9. Conclusion

Project 002 successfully deployed a locally hosted Qwen3 4B language model within an isolated Ubuntu Server virtual machine on Proxmox VE. The deployment demonstrated practical application of Linux administration, logical volume management, virtual machine resource allocation, service configuration and network security controls.

Validation confirmed the expanded storage and memory allocations, operational status of the Ollama service, enforcement of the tested outbound firewall restriction and inaccessibility of the API from the tested remote laboratory host. Performance benchmarking also demonstrated the benefits and limitations of increasing CPU resources for local inference.

Although the deployment met its primary functional and security objectives, the available hardware imposed significant performance constraints. Future development may investigate alternative model sizes, additional processing resources and controlled automation capabilities, subject to separate security assessments.

The project established a documented baseline for operating a local language model within a security-controlled virtualised environment and demonstrated the importance of validating both system functionality and the controls intended to protect it.




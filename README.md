# Hybrid AI-Driven SOC Lab

Welcome to the **Next-Gen Hybrid AI-Driven SOC Lab**. This project is a state-of-the-art, multi-cloud, and deeply automated home lab designed to simulate a modern Enterprise Security Operations Center (SOC). 

It integrates Next-Gen detection technologies, Active Deception, continuous adversary emulation, and an **Autonomous AI Agent** (Local LLM) for intelligent triage and dynamic rule generation.

## Architecture & Network Topology

Here is the high-level architecture of the lab, demonstrating network segmentation, multi-cloud log ingestion, and the AI automation pipeline:

```mermaid
graph TD
    %% Adversary & Attack Simulation
    subgraph Red_Team [Adversary & Emulation Zone]
        Attacker[Kali Linux - Manual Attacks]
        Caldera[CALDERA / Atomic Red Team - Automated Emulation]
    end
    Attacker -->|VPN / External| FW[pfSense Firewall / Router]
    Caldera -->|Continuous Simulated Attacks| FW
    
    %% Multi-Cloud Environment
    subgraph Multi_Cloud [Multi-Cloud Log Sources]
        subgraph AWS [AWS Cloud]
            AWS_Logs[Route 53, VPC, CloudTrail] --> AWS_S3[S3 Bucket] --> AWS_SQS[SQS]
        end
        subgraph Azure [Microsoft Azure]
            Azure_Logs[Activity, Entra ID] --> Azure_EH[Event Hubs]
        end
        subgraph GCP [Google Cloud Platform]
            GCP_Logs[GCP Services Logging] --> GCP_PubSub[Cloud Pub/Sub]
        end
    end
    
    %% Cloud Log Ingestion Pipeline
    AWS_SQS -.->|aws.yml| FileBeat
    Azure_EH -.->|azure.yml| FileBeat
    GCP_PubSub -.->|gcp.yml| FileBeat
    
    %% DMZ Subnet
    FW -->|VLAN 10| DMZ{DMZ Subnet}
    subgraph DMZ_Zone [DMZ Zone - The Trap]
        DMZ --> Honeypot[Traditional Honeypot]
        DMZ --> DVWA[Vulnerable Web Apps]
    end
    
    %% Victim Subnet with Next-Gen Defense
    FW -->|VLAN 20| Victim{Victim Subnet}
    subgraph Victim_Zone [Victim Network - Target]
        Victim --> AD[Active Directory / DC]
        Victim --> Win[Windows Clients]
        Victim --> Linux[Linux Servers]
        
        %% Next-Gen Tech Added
        Win -.-> Honeytokens[CanaryTokens - Active Deception]
        Linux -.-> eBPF[Tetragon / Falco - eBPF Kernel Security]
    end
    
    %% SOC & Monitoring Subnet
    FW -->|VLAN 30 + SPAN Port| SOC{SOC Subnet}
    subgraph SOC_Zone [SOC & Defense Center]
        FileBeat[FileBeat - Log Shipper] --> Elastic[ElasticSearch]
        Elastic --> Monitoring[Monitoring Dashboard / Kibana]
        SOC --> Wazuh[Wazuh SIEM / XDR]
        SOC --> SecOnion[Security Onion - Network Monitor]
        SOC --> SOAR[Shuffle SOAR - Orchestration]
        SOC --> TheHive[The Hive - Incident Response]
    end
    
    %% Telemetry Routing
    eBPF -.->|Deep Runtime Logs| Elastic
    Honeytokens -.->|Zero-FP Alerts| Wazuh
    
    %% Advanced Autonomous AI Flow
    Wazuh -- 1. Triggers Alert --> SOAR
    SecOnion -- 1. Triggers Alert --> SOAR
    Elastic -- 1. Triggers Alert --> SOAR
    
    SOAR -- 2. Sends Context --> AI((Autonomous AI Agent - Ollama))
    AI -- 3a. Triage & Explanation --> SOAR
    AI -- 3b. Generates Sigma/YARA Rules --> SOAR
    
    SOAR -- 4a. Auto-Creates Ticket --> TheHive
    SOAR -- 4b. Deploys Defense Rules --> Wazuh```

Technology Stack & ToolsInfrastructure & Routing: pfSense, Proxmox/VMware, VLAN Segmentation.SIEM & Log Aggregation: Wazuh, Elastic Stack (Filebeat, Elasticsearch, Kibana).Network Security Monitoring: Security Onion, Zeek/Suricata.Next-Gen Security: Tetragon/Falco (eBPF Kernel Monitoring).Active Deception: CanaryTokens (Honeytokens).SOAR & Incident Response: Shuffle, TheHive.AI & Automation: Local LLMs (Ollama) as a Tier-2 Analyst.Adversary Emulation: Kali Linux, MITRE CALDERA, Atomic Red Team.

Resource Allocation (Hardware Engineering)To ensure smooth operation without overloading the host machine, resources are strictly allocated utilizing headless VMs and Docker containers. The total maximum RAM footprint is optimized to ~18-20 GB.Component / NodeEnvironmentRAMvCPUNotespfSense (Firewall/Router)VMware VM1 GB1Lightweight, handles routing across 4 VLANs.SOC & Defense CenterUbuntu Server VM + Docker8 - 10 GB4The core engine. Hosts Docker containers for Wazuh, Elastic, Shuffle, and TheHive.Victim: Active DirectoryWindows Server Core VM2 GB2Headless version (No GUI) to save resources.Victim: Windows ClientWindows 10/11 VM4 GB2Main target endpoint for attack simulation.DMZ: Vulnerable WebLinux VM (Micro)1 GB1Hosts intentionally vulnerable apps (e.g., DVWA).Adversary: Kali LinuxLinux VM2 GB2Used for manual attacks and managing CALDERA.AI Agent (Ollama)Host OS (Windows 11)0 GB (VM)N/A

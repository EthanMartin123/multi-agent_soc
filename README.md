# multi-agent_soc
This capstone builds a multi-agent AI SOC team that triages, enriches, investigates, and reports on alerts

# Team Members
Ethan Martin

Jared Sagmaquen

# Research Question
Can specialized agents improve the speed and consistency of alert investigations compared with one general AI agent?

# Our Approach
AI agents dividing the workflow of triage, investigation, and response plan into focused roles while sharing one incident record.

**Triage Agent**: Will classify the alert. Set severity and priority.

**Investigation Agent**: Searches related logs. Builds an evidence timeline.

**Response Agent**: Check response playbooks, and recommends an action.

**Human Analyst**: Reviews the evidence. Approves or rejects the response.

# Planned Technologies, Tools, or Programming Languages

### Lab Infrastructure

**Proxmox VE**: Hypervisor

**GOAD**: Vulnerable AD Environment. Deployment uses **Terraform** and **Ansible**.

**Sysmon**: Rich endpoint events, plus native Windows event logs

**Wazuh**: SIEM

**Zeek**: network telemetry

**Sigma rules**: for rule-based baseline to compare the agents against

**Python**: chosen programming language for this project

**OpenClaw**: as the agent framework

**Claude**: our chosen LLM for this project

**Sliver**: Command & Control framework

**Git**: version control

**Docker**: containerizing the agent stack for clean teardown and portability

**Jupyter**: to show detection-rate and time-to-detect metrics in charts for the writeup

# Capstone Project Timeline

## Phase 1 — Planning & Design
- Finalize scope, success metrics, and the agent architecture
- Confirm hardware budget, licensing, and school IP requirements
- Draft the project proposal
- 
## Phase 2 — Lab Infrastructure
- Stand up Proxmox, isolated VLAN, and firewall
- Deploy GOAD (start with GOAD-Light) and verify the AD environment
- Establish teardown-and-rebuild-from-scripts workflow
- 
## Phase 3 — Telemetry & SIEM
- Deploy Sysmon and ship Windows event logs
- Stand up the SIEM (Wazuh/Elastic) and confirm ingestion
- Add network telemetry (Zeek/Suricata) and a Sigma rule baseline
- 
## Phase 4 — Agent Development
- Build the shared case object and agent scaffolding in OpenClaw
- Implement agents one at a time: triage, enrichment, investigation, response, reporting
- Wire up the human approval gate and least-privilege tooling
- 
## Phase 5 — Adversary Emulation & Testing
- Script a repeatable Sliver attack chain mapped to MITRE ATT&CK
- Run the agent team against the baseline across multiple transports/profiles
- Collect detection rate, time-to-detect, and false-positive metrics
- 
## Phase 6 — Analysis & Writeup
- Crunch metrics into charts and compare agents vs. baseline
- Write the final report and prepare the presentation/demo
- Clean up the repo, docs, and README
- 
## Buffer
- Reserve time at the end for troubleshooting, rework, and the inevitable thing that breaks the week before the deadline.

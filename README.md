# Human-Supervised Multi-Agent SOC for Security Alert Investigation

This capstone builds a prototype Security Operations Center (SOC) in which specialized AI agents investigate security alerts and a human analyst approves every response. It then tests whether those agents do better than one general AI agent and a rule-based baseline.

COMP/IT 491 and 499 Capstone, Department of Computer Science, California State University Channel Islands
Adviser: Dr. Reza Abdolee

## Team Members

- Ethan Martin
- Jared Sagmaquen

## Project Overview

A SOC watches an organization's systems for signs of attack. Security tools send alerts, and an analyst reads each one, decides if it matters, searches logs to learn what happened, and chooses a response.

This project builds an isolated lab, runs scripted attacks inside it, and sends the resulting alerts to a team of AI agents built with OpenClaw. A manager agent is the analyst's single point of contact. It hands work to three specialist agents (triage, investigation, and response), each with one job and a short list of tools. A central focus of the project is keeping each agent on its intended role so that no agent takes on another agent's task. The agents investigate and recommend, and a person decides. The prototype recommends responses only and does not change any real network. It is a research prototype, not a production tool.

## Problem Statement

SOC analysts receive more alerts than they can investigate, and many are false alarms. Checking them by hand is tedious and can lead to burnout and to analysts ignoring alarms. The evidence for one alert is spread across endpoint logs, account records, and network data, and a wrong response, such as blocking an account or isolating a computer, can stop legitimate work.

Generative AI may help, but there is little open evidence on how to design these systems. It is not known whether specialized agents with different tools beat one general agent, or whether any gain is worth the extra cost. We found no open study that compares a multi-agent design, a single-agent design, and a rule baseline on the same lab incidents while tracking cost and unsupported claims next to accuracy. This project aims to fill that gap on a small scale.

## Research Question

Can specialized agents improve the speed and consistency of security alert investigations compared with one general AI agent?

## Goals and Objectives

By the end of COMP/IT 499 we aim to:

1. **Build the lab.** An isolated lab with a Windows domain, endpoint and network logging, and a SIEM, all scripted so it can be rebuilt.
2. **Build the agent team.** A manager agent that the analyst works with, and three specialist agents under it (triage, investigation, and response) that share one case record.
3. **Keep agents in their roles.** Give each agent one task and only the tools and permissions that task needs, so no agent works on more than one part of the investigation.
4. **Keep a person in control.** No response happens without a human analyst's approval.
5. **Build the comparisons.** A single-agent version using the same model and tools, and a Sigma rule baseline.
6. **Create test scenarios.** Labeled attack scenarios mapped to MITRE ATT&CK, each with an answer key.
7. **Measure and report.** Speed, accuracy, consistency, unsupported claims, and cost for each system, reported with charts.
8. **Document the work.** Keep this repository up to date with code, setup steps, and test results.

Stretch goals, if time allows:

- Add an enrichment agent and a reporting agent.
- Test whether agents follow instructions hidden inside log text.

## System Components

### Agents

The agent team runs in OpenClaw. The analyst talks only to the manager agent. The manager hands each step to the right specialist, passes structured data (JSON) between them, and returns the finished case to the analyst. Every step is saved in a shared case record.

```
Human Analyst
     |
Manager Agent
     |
     +-- Triage Agent
     +-- Investigation Agent
     +-- Response Agent
```

| Component | Job | Planned tools |
|---|---|---|
| Manager Agent | Receives alerts and analyst requests, assigns work to the specialist agents in order, tracks the case, and presents the evidence and recommended response to the analyst. Does not triage, investigate, or recommend on its own. | Assign task, read and update the case record |
| Triage Agent | Classifies the alert, sets severity and priority, and lists the evidence it needs | Alert lookup, risk score |
| Investigation Agent | Searches related logs, finds linked events, users, devices, and IP addresses, and builds a timeline | Search events by IP address, find related logins, build a timeline |
| Response Agent | Reviews the findings, checks approved playbooks, and recommends an action | Playbook search, draft a recommendation |
| Human Analyst | Works with the manager agent, reviews the evidence, and approves or rejects the recommended response | Dashboard |

### Role Separation

Keeping each agent on its own task is a core part of the design:

- **One job per agent.** Each agent has its own instructions describing its single task and what is outside its scope.
- **Separate tools.** Each agent has its own OpenClaw workspace, sandbox, and tool allow and deny lists, so it cannot call tools that belong to another role. For example, the triage agent cannot search logs, and the investigation agent cannot search playbooks.
- **Defined handoffs.** Agents receive work only from the manager and return a fixed output format for their step. Output that goes beyond the agent's role is rejected.
- **Manager does not do the work.** The manager routes and tracks tasks but has no investigation or response tools of its own.
- **Logged and checked.** Every tool call is recorded with the agent that made it, so out-of-role activity can be found and counted.

### Comparison Systems

- **Single-agent version:** one general agent with the same model and tools.
- **Sigma rule baseline:** fixed detection rules, no agents. Compared on detection measures only.

### Safety and Reliability

- **Evidence for every claim.** Each important claim must point to a log event. A validation step checks that cited events exist and flags any claim without one as unsupported.
- **Structured output.** Agent output follows a set format. Malformed output is rejected with a limited number of retries.
- **Least privilege.** Search tools are read-only, and no agent can change a system. Every tool call and decision is written to an audit log.
- **Untrusted framework.** The agent stack runs in containers on the isolated network, with no third-party plugins and with shell, file-write, and browser tools turned off unless a test needs them.
- **Untrusted log text.** Agents treat log text as evidence and ignore any instructions in it.

### Evaluation Measures

| Measure | Systems compared |
|---|---|
| Classification accuracy | Multi-agent, single agent |
| Severity accuracy | Multi-agent, single agent |
| Evidence quality | Multi-agent, single agent |
| Unsupported claims | Multi-agent, single agent |
| Response correctness | Multi-agent, single agent |
| Consistency across repeat runs | Multi-agent, single agent |
| Role adherence (tool calls and outputs that stay within each agent's role) | Multi-agent |
| Detection rate | All three |
| False positives | All three |
| Time to detect | All three |
| Investigation time | Multi-agent, single agent |
| Cost (tokens and tool calls per case) | Multi-agent, single agent |

The test set is small, so results will be reported as counts and ranges without statistical tests.

## Planned Technologies and Tools

### Lab Infrastructure

- **Proxmox VE:** hypervisor
- **Terraform and Ansible:** scripted build, teardown, and rebuild
- **GOAD-Light:** three-machine Windows Active Directory practice lab from Orange Cyberdefense
- **Sysmon:** detailed process, network, and file events, plus native Windows event logs
- **Zeek:** network activity records
- **Wazuh:** SIEM for collecting logs and raising alerts
- **Sigma rules:** rule-based baseline to compare the agents against
- **Sliver:** planned open-source adversary emulation framework for repeatable attack scenarios
- **MITRE ATT&CK:** shared names for attacker tactics and techniques in scenarios and answer keys

### Agent Stack

- **OpenClaw:** agent framework for the manager and specialist agents, using per-agent workspaces, sandboxes, and tool allow and deny lists
- **Python:** agent tools, validation, and analysis scripts
- **Claude:** the LLM (no model training)
- **Docker:** containerized agent stack for clean teardown and portability

### Analysis and Project Management

- **Jupyter:** charts for speed, accuracy, consistency, and cost results
- **Git and GitHub:** version control and documentation

## Implementation Plan and Timeline

By the end of COMP/IT 491 we aim to have the lab running, logs flowing into the SIEM, and a first triage agent working on one scenario. Dates are approximate and will be updated as we learn more.

### Phase 1: Planning and Design (Oct to Nov 2026, COMP/IT 491)

- Set scope and success measures
- Choose the agent design
- Confirm hardware and school requirements
- Write the project proposal

### Phase 2: Lab Infrastructure (Nov to Dec 2026, COMP/IT 491)

- Set up Proxmox, the isolated network, and the firewall
- Deploy GOAD-Light and verify the Active Directory environment
- Script teardown and rebuild

### Phase 3: Logging and SIEM (Dec 2026 to Jan 2027, COMP/IT 491 and 499)

- Add Sysmon, Windows event logs, and Zeek
- Set up Wazuh and confirm logs arrive
- Write the Sigma baseline rules

### Phase 4: Agent Development (Dec 2026 to Mar 2027, COMP/IT 491 and 499)

- Set up OpenClaw and the shared case record
- Build the manager agent and its handoff to the analyst
- Build the triage, investigation, and response agents one at a time, each with its own workspace and tool limits
- Add the human approval step and role-separation checks
- Build the single-agent version

### Phase 5: Scenarios and Testing (Mar to Apr 2027, COMP/IT 499)

- Write answer keys for each scenario
- Script and run attack scenarios mapped to MITRE ATT&CK, starting with suspicious sign-in activity followed by lateral movement
- Run all three systems several times per scenario and collect data

### Phase 6: Analysis and Writeup (Apr to May 2027, COMP/IT 499)

- Chart the results and compare the systems
- Write the final report
- Prepare the presentation and demo

### Buffer (May 2027, COMP/IT 499)

- Time for repairs and rework

## Expected Challenges

- **Answers vary.** LLM output changes between runs. Repeated runs, fixed prompts, and low randomness settings will help measure and limit this.
- **Lab setup time.** Starting with GOAD-Light, scripting every build, and keeping a schedule buffer.
- **Small test set.** A few scenarios limit what the results can show, and the final report will say so.
- **Role drift.** Agents may try to do work outside their role. Tool limits, fixed output formats, and logged tool calls will help prevent and measure this.
- **Framework security.** Published research has found security weaknesses in OpenClaw, so the agent stack runs in containers on the isolated network with no third-party plugins.

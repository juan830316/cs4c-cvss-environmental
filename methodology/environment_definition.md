# Environment Definition

## Overview

This document formally defines the target environment used for the CS4C contextualization study. The environment represents a typical **medium-sized enterprise infrastructure** with high availability requirements, segmented networks, and a mix of modern and legacy systems.

## Target Infrastructure Profile

### Network Topology

- **DMZ Segment**: Public-facing web servers, API gateways, and reverse proxies.
- **Internal Corporate Network**: Workstations, file servers, identity management systems.
- **Restricted Zone**: Database servers, backup infrastructure, sensitive data repositories.
- **Legacy Segment**: End-of-life operating systems and applications that cannot be patched due to vendor discontinuation or operational constraints.

### Asset Criticality Classification

Each asset class is assigned a CIA requirement level (Confidentiality, Integrity, Availability) on a scale of Low, Medium, or High, following the CVSS v3.1 specification for environmental metrics.

| Asset Class                | Confidentiality Requirement | Integrity Requirement | Availability Requirement |
|----------------------------|-----------------------------|-----------------------|--------------------------|
| Web Server (DMZ)           | Medium                      | High                  | High                     |
| Database Server            | High                        | High                  | High                     |
| Internal Workstation       | Medium                      | Medium                | Low                      |
| Legacy Application Server  | Low                         | Low                   | High                     |
| Backup Infrastructure      | High                        | High                  | Medium                   |
| Network Appliance          | Low                         | Medium                | High                     |

### CIA Requirement Weights

Per CVSS v3.1 specification, the requirement weights used in the environmental score calculation are:

| Requirement Level | Weight (CR/IR/AR) |
|-------------------|-------------------|
| Low               | 0.5               |
| Medium            | 1.0               |
| High              | 1.5               |

## Environmental Metrics Configuration

### Modified Base Metrics

The following modified base metrics are applied to reflect network-specific constraints:

| Metric | Symbol | Value | Rationale |
|--------|--------|-------|-----------|
| Modified Attack Vector     | MAV | N (Network) or A (Adjacent) | DMZ services are network-accessible; legacy segment is air-gapped |
| Modified Attack Complexity | MAC | L (Low) or H (High)        | Internal services: Low; legacy: High due to access controls |
| Modified Privileges Required | MPR | N (None) or L (Low)       | Public endpoints: None; internal: Low (authentication required) |
| Modified User Interaction  | MUI | N (None) or R (Required)   | Automated services: None; administrative consoles: Required |
| Modified Scope             | MS  | U (Unchanged) or C (Changed) | Defined per CVE based on impact analysis |
| Modified Confidentiality   | MC  | H (High), L (Low), N (None) | Inherited from base vector |
| Modified Integrity         | MI  | H (High), L (Low), N (None) | Inherited from base vector |
| Modified Availability      | MA  | H (High), L (Low), N (None) | Inherited from base vector |

### Rationale for Modifications

1. **MAV set to Adjacent for legacy segment**: Legacy systems are isolated in a separate VLAN with no direct Internet exposure, reducing the attack vector from Network to Adjacent.

2. **MAC set to High for internal services**: Additional authentication layers and network segmentation increase the complexity of exploiting these services from the outside.

3. **CR/IR set to High for database servers**: These servers store sensitive operational data; both confidentiality and integrity are critical.

4. **AR set to High for web servers and legacy systems**: Web servers face external traffic and require high uptime; legacy systems often support critical production workflows.

## Software and System Inventory

The target environment includes the following software components — each potentially hosting one or more of the selected CVEs:

- **Operating Systems**:
  - Windows Server 2016/2019 (patched)
  - Windows Server 2008 R2 (legacy, unpatched)
  - Ubuntu 18.04 LTS (patched)
  - CentOS 6 (legacy, unpatched)

- **Applications**:
  - Apache HTTP Server 2.4.x (patched)
  - Apache Tomcat 7.x (legacy, partially unpatched)
  - Microsoft IIS 8.5 (patched)
  - OpenSSL 1.0.x (legacy, unpatched)

- **Middleware & Databases**:
  - MySQL 5.7 (patched)
  - PostgreSQL 9.6 (legacy, partially unpatched)
  - Redis 4.x (patched)

## Threat Model Assumptions

1. **Attacker Profile**: External attacker with low sophistication (commodity malware, known exploit kits) and internal attacker with authenticated access to the corporate network.

2. **Attack Surface**: Public-facing web services (DMZ), internal corporate applications, and legacy systems with limited monitoring.

3. **Assumptions about Patching**:
   - All modern systems receive security patches within the vendor-defined window.
   - Legacy systems cannot be patched due to vendor end-of-life or operational continuity requirements.
   - Virtual patching (IDS/IPS, WAF rules) is in place for some legacy vulnerabilities.

## Applicability and Limitations

- This environment definition is representative but not exhaustive. The CS4C methodology is designed to be adaptable to other environment profiles by adjusting the CIA requirement weights and modified base metrics accordingly.
- The environmental score is sensitive to the accuracy of the CIA requirement classification. Misclassification may lead to over- or under-prioritization.
- The environment definition should be revisited whenever the infrastructure undergoes significant changes (e.g., migration, decommissioning of legacy systems).

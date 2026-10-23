# Nmap Technical Reference & Network Reconnaissance

## 1. Overview

Nmap (Network Mapper) is an open-source tool used for network discovery, port scanning, service identification, and security auditing.

It helps security professionals understand which hosts are reachable, which network services are exposed, and what information can be gathered from a system during an authorized assessment.

**Primary applications:**
- Network discovery and asset inventory
- TCP and UDP port scanning
- Service and version detection
- Operating system fingerprinting
- Network security auditing
- Script-assisted service enumeration
- Validation of network exposure

## 2. Core Networking Concepts

### IP Address
A logical address used to identify a network interface within an IP network.

### Port
A numbered communication endpoint associated with a transport-layer protocol. TCP and UDP have separate port spaces.

### Service
An application or process that communicates through a network endpoint, such as a web server or SSH server.

### TCP
A connection-oriented transport protocol that provides reliable, ordered data delivery.

### UDP
A connectionless transport protocol that does not provide TCP-style delivery guarantees.

### Host Discovery
The process of determining which target systems appear reachable through network probes.

### Port Scanning
The process of probing selected ports to infer their current states.

## 3. Nmap Scan Workflow

A typical Nmap assessment may involve these stages:

1. Define the authorized target scope.
2. Discover reachable hosts.
3. Identify relevant TCP and UDP ports.
4. Determine port states.
5. Identify services and versions.
6. Perform appropriate additional enumeration.
7. Save and analyze scan results.
8. Document findings and limitations.

Not every scan requires every stage. The workflow depends on the assessment objective.

## 4. Target Specification

Examples of target formats:

```bash
nmap 192.168.1.10
nmap 192.168.1.10 192.168.1.20
nmap 192.168.1.0/24
nmap -iL targets.txt
```

- A single IP address specifies one target.
- Multiple addresses specify individual targets.
- CIDR notation specifies an IP network range.
- `-iL` reads targets from a file.

Only use target ranges that are explicitly authorized.

## 5. Host Discovery

Host discovery helps determine which systems appear reachable.

```bash
nmap -sn 192.168.1.0/24
```

The `-sn` option performs host discovery without a conventional port scan.

```bash
nmap -Pn 192.168.1.10
```

The `-Pn` option skips the usual host-discovery stage and treats the target as online for subsequent scanning.

Important considerations:
- A host may be online but fail to respond to discovery probes.
- Firewalls can affect probe responses.
- Network location and routing influence scan visibility.
- Host discovery results are not a guarantee of actual system availability.

## 6. Port Scanning

### Default Scan

```bash
nmap 192.168.1.10
```

Performs a basic scan of the target's commonly scanned TCP ports.

### TCP SYN Scan

```bash
sudo nmap -sS 192.168.1.10
```

Uses TCP SYN probes to infer port states. This scan generally requires raw-packet privileges.

### TCP Connect Scan

```bash
nmap -sT 192.168.1.10
```

Uses the operating system's connect functionality to establish TCP connections.

### UDP Scan

```bash
sudo nmap -sU 192.168.1.10
```

Probes UDP ports. Results may be slower or less conclusive because many UDP services do not respond to empty or unexpected probes.

### Selecting Ports

```bash
nmap -p 22,80,443 192.168.1.10
nmap -p- 192.168.1.10
```

- `-p` selects specific ports.
- `-p-` selects all TCP ports from 1 through 65535 for a standard TCP scan.

## 7. Understanding Port States

Nmap commonly reports these states:

| State | Meaning |
|---|---|
| Open | An application is accepting connections or responding as an open service. |
| Closed | The port is reachable, but no application is listening on it. |
| Filtered | Filtering or lack of a conclusive response prevents Nmap from determining whether the port is open. |
| Unfiltered | The port is accessible to the probes, but its open or closed state is not determined by that scan type. |
| Open\|Filtered | Nmap cannot distinguish between an open port and a filtered port. |
| Closed\|Filtered | Nmap cannot distinguish between a closed port and a filtered port. |

Port states describe what Nmap can infer from its probes. They do not independently establish that a service is vulnerable.

## 8. Service and Version Detection

```bash
nmap -sV 192.168.1.10
```

The `-sV` option probes detected ports to identify the services and, where possible, their versions.

This information can help analysts:
- Inventory exposed services.
- Understand which applications are reachable.
- Identify software requiring further verification.
- Prioritize follow-up security checks.

Version detection may be inconclusive. Service banners can be hidden, modified, or misleading, and detected versions should be verified before drawing security conclusions.

## 9. Operating System Detection

```bash
sudo nmap -O 192.168.1.10
```

The `-O` option attempts to infer the target operating system by analyzing network responses and comparing them with known characteristics.

Accuracy can be affected by:
- Firewalls and packet filtering
- Virtualization and network devices
- Incomplete responses
- Similar network-stack behavior across operating systems

OS detection is an estimate, not definitive proof.

## 10. Nmap Scripting Engine (NSE)

The Nmap Scripting Engine extends Nmap with scripts for service discovery, information gathering, and other network assessment tasks.

```bash
nmap -sC 192.168.1.10
```

The `-sC` option runs the default NSE script set.

```bash
nmap --script-help default
```

Displays help for the default script category.

NSE scripts have different purposes and risk levels. Review a script's behavior before running it, and use scripts only within the authorized assessment scope.

## 11. Timing and Performance

Nmap provides timing templates that influence scan speed and network load.

```bash
nmap -T3 192.168.1.10
nmap -T4 192.168.1.10
```

- `-T3`: Normal timing template; this is the default.
- `-T4`: More aggressive timing intended for reasonably reliable networks.

Faster settings do not guarantee better results. Packet loss, latency, filtering, and rate limiting can affect both speed and accuracy.

Use conservative settings when the target environment or its capacity is uncertain.

## 12. Output and Documentation

Nmap supports multiple output formats.

```bash
nmap -oN scan.txt 192.168.1.10
nmap -oX scan.xml 192.168.1.10
nmap -oG scan.gnmap 192.168.1.10
nmap -oA results 192.168.1.10
```

- `-oN`: Normal human-readable output.
- `-oX`: XML output for structured processing.
- `-oG`: Grepable output, a legacy format.
- `-oA`: Saves output in the principal supported formats using a shared filename prefix.

Saved results should be reviewed and handled according to the lab's data-handling requirements.

## 13. Useful Diagnostic Options

```bash
nmap --version
nmap -v 192.168.1.10
nmap --reason 192.168.1.10
nmap --packet-trace 192.168.1.10
```

- `--version`: Displays the installed Nmap version.
- `-v`: Increases output verbosity.
- `--reason`: Shows the reason Nmap assigned a reported state.
- `--packet-trace`: Displays sent and received packets for troubleshooting.

Packet traces can produce extensive output. Use them when investigating scan behavior, rather than as a default for every scan.

## 14. Interpreting Findings

An open port is not automatically a vulnerability.

A responsible assessment considers:
- Whether the service is expected to be exposed.
- Whether the service and version identification is reliable.
- Whether the service is appropriately configured.
- Whether access controls are in place.
- Whether additional validation is necessary.
- Whether the finding is within the assessment scope.

Nmap supports reconnaissance and enumeration. Additional evidence is required to confirm most security vulnerabilities.

## 15. Common Limitations

- Firewalls can hide or alter responses.
- Packet loss can produce incomplete results.
- UDP scans may take considerable time.
- Version detection may misidentify services.
- OS detection may be uncertain.
- A successful scan does not prove that a system is secure.
- Scan results represent observations from a particular time and network position.

## 16. Practical Lab Report

Complete this section with findings from an authorized lab.

**Lab platform:** [Name of platform or local lab]

**Date:** [Date of assessment]

**Scope:** [Authorized target or lab identifier]

**Objective:** [What the scan was intended to determine]

**Nmap version:** [Output of nmap --version]

**Commands executed:** [Commands actually used]

**Host discovery results:** [Observed results]

**Port states:** [Observed results]

**Services and versions:** [Observed results]

**Interpretation:** [Explain what the results mean]

**Limitations:** [Describe uncertainty or incomplete results]

**Lessons learned:** [Your own technical conclusions]

## 17. Professional and Ethical Considerations

- Obtain explicit authorization before scanning systems.
- Keep testing within the defined scope.
- Respect rate limits and assessment rules.
- Avoid disruptive testing without specific permission.
- Protect scan output that contains sensitive infrastructure details.
- Report findings accurately and distinguish observations from confirmed vulnerabilities.

## 18. References

- Official Nmap Documentation:
  https://nmap.org/docs.html

- Nmap Reference Guide:
  https://nmap.org/book/man.html

- Official Nmap Network Scanning Book:
  https://nmap.org/book/toc.html

---

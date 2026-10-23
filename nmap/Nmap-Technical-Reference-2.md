## 5. TCP Flags and Connection Mechanics

TCP flags are control bits in the TCP header that help manage connection establishment, data transfer, and connection termination.

### 5.1 Important TCP Flags

| Flag | Full Name | Purpose |
|---|---|---|
| SYN | Synchronize | Initiates a TCP connection and synchronizes sequence numbers. |
| ACK | Acknowledgment | Acknowledges received TCP data or connection-related information. |
| RST | Reset | Aborts or rejects a connection. |
| FIN | Finish | Indicates that a sender has finished sending data. |
| PSH | Push | Requests that buffered data be delivered to the receiving application promptly. |
| URG | Urgent | Indicates that the urgent pointer field is significant. |
| ECE | ECN-Echo | Participates in Explicit Congestion Notification. |
| CWR | Congestion Window Reduced | Indicates that the sender has responded to a congestion notification. |
| NS | Nonce Sum | An additional ECN-related flag defined in TCP specifications. |

### 5.2 TCP Three-Way Handshake

A normal TCP connection is established through three steps:

1. **SYN:** The client requests a connection.
2. **SYN-ACK:** The server acknowledges the request and sends its own SYN.
3. **ACK:** The client acknowledges the server's response.

After this exchange, the TCP connection is established.

### 5.3 TCP Connection Termination

A typical orderly TCP connection termination uses FIN and ACK exchanges to close the connection in each direction.

RST is different: it resets a connection rather than performing an orderly shutdown.

### 5.4 Why TCP Flags Matter in Nmap

Nmap can use TCP responses to infer port states.

- A SYN-ACK response commonly indicates an open TCP port during SYN scanning.
- A RST response commonly indicates a closed TCP port.
- No response or certain ICMP error responses may cause Nmap to classify a port as filtered, depending on the scan type and context.

These are interpretations of network responses, not absolute guarantees. Firewalls and unusual network configurations can affect the results.

## 6. Port Scanning Techniques

Port scanning probes TCP or UDP ports to determine their states and identify potentially exposed services.

### 6.1 TCP SYN Scan

```bash
sudo nmap -sS 192.168.1.10
```

A SYN scan sends TCP SYN probes without completing a normal TCP connection when it receives the expected response.

Typical interpretations:

- SYN-ACK: Open
- RST: Closed
- No response or certain network errors: Filtered or otherwise inconclusive

SYN scanning is commonly used when the required packet privileges are available.

### 6.2 TCP Connect Scan

```bash
nmap -sT 192.168.1.10
```

This scan uses the operating system's connect function to establish TCP connections.

It is useful when raw-packet scanning privileges are unavailable.

### 6.3 UDP Scan

```bash
sudo nmap -sU 192.168.1.10
```

UDP does not use the TCP handshake or TCP flags.

Nmap interprets UDP responses and certain ICMP errors to infer port states. A lack of response often produces an open|filtered result, so additional validation may be needed.

### 6.4 TCP ACK Scan

```bash
sudo nmap -sA 192.168.1.10
```

An ACK scan helps examine packet-filtering behavior.

- RST responses generally indicate an unfiltered port.
- Lack of a response or certain ICMP errors can indicate filtering.

An ACK scan does not directly determine whether a TCP service is open or closed.

### 6.5 TCP FIN Scan

```bash
sudo nmap -sF 192.168.1.10
```

A FIN scan sends TCP packets with the FIN flag set.

On systems following conventional TCP behavior, closed ports commonly respond with RST, while open ports may not respond.

Results can vary by operating system and firewall behavior.

### 6.6 TCP NULL Scan

```bash
sudo nmap -sN 192.168.1.10
```

A NULL scan sends TCP packets without setting any TCP control flags.

Its interpretations rely on TCP behavior and may be affected by filtering and operating-system differences.

### 6.7 TCP Xmas Scan

```bash
sudo nmap -sX 192.168.1.10
```

An Xmas scan sets FIN, PSH, and URG flags.

As with FIN and NULL scans, responses depend on the target's TCP implementation and network filtering.

### 6.8 TCP Window Scan

```bash
sudo nmap -sW 192.168.1.10
```

A Window scan examines TCP RST responses and the TCP window field to attempt to distinguish port states on certain systems.

It is not reliable on all operating systems.

### 6.9 Maimon Scan

```bash
sudo nmap -sM 192.168.1.10
```

A Maimon scan uses TCP FIN and ACK flags. Its results depend on the target's implementation and may be inconclusive.

### 6.10 Selecting Ports

```bash
nmap -p 22,80,443 192.168.1.10
nmap -p- 192.168.1.10
nmap -p 1-1024 192.168.1.10
```

- `-p 22,80,443`: Scans selected ports.
- `-p-`: Scans TCP ports 1–65535 when used with a TCP scan.
- `-p 1-1024`: Scans the specified port range.

### 6.11 Comparing Scan Techniques

| Technique | Option | Primary Purpose |
|---|---|---|
| SYN | `-sS` | TCP port-state discovery |
| Connect | `-sT` | TCP scanning using full connections |
| UDP | `-sU` | UDP port-state discovery |
| ACK | `-sA` | Examining filtering behavior |
| FIN | `-sF` | TCP flag-based probing |
| NULL | `-sN` | TCP flag-based probing |
| Xmas | `-sX` | TCP flag-based probing |
| Window | `-sW` | Examining TCP window responses |
| Maimon | `-sM` | TCP FIN/ACK-based probing |

These methods differ in their probes and interpretations. None guarantees that a port's true state will always be determined correctly.

## 7. Post-Port-Scan Enumeration

Post-port-scan enumeration is the process of investigating discovered ports and services to understand what is running and whether further assessment is justified.

### 7.1 Identify Services and Versions

```bash
nmap -sV -p 22,80,443 192.168.1.10
```

Attempts to identify services and versions on selected ports.

### 7.2 Run Default NSE Scripts

```bash
nmap -sC -p 22,80,443 192.168.1.10
```

Runs Nmap's default NSE script set against the specified target and ports.

Review script behavior and target authorization before use.

### 7.3 Combine Service Detection and Default Scripts

```bash
nmap -sC -sV 192.168.1.10
```

Combines service/version detection with the default NSE scripts.

### 7.4 Examine Specific Ports

```bash
nmap -sV -p 22 192.168.1.10
nmap -sV -p 80,443 192.168.1.10
```

Investigates particular services based on previously observed scan results.

### 7.5 HTTP Service Enumeration

For an authorized lab with a discovered web service:

```bash
nmap -p 80,443 --script http-title 192.168.1.10
```

The `http-title` script attempts to retrieve webpage titles from HTTP services.

This can provide initial context about the web application but does not prove that a vulnerability exists.

### 7.6 SSH Service Enumeration

```bash
nmap -p 22 --script ssh2-enum-algos 192.168.1.10
```

Attempts to enumerate supported SSH algorithms on an accessible SSH service.

Results can help assess protocol configuration, but they require careful interpretation.

### 7.7 Save and Review Results

```bash
nmap -sV -oA nmap-results 192.168.1.10
```

Saves scan output in multiple formats for later review and documentation.

### 7.8 Post-Scan Analysis Workflow

1. Review discovered open ports.
2. Identify associated services and versions.
3. Determine whether the services are expected to be exposed.
4. Select appropriate enumeration techniques.
5. Record observations and supporting evidence.
6. Verify potential issues before reporting them.
7. Document limitations and recommendations.

Post-scan enumeration should remain within the authorized scope. It does not automatically imply exploitation or vulnerability confirmation.

## 8. TCP Flag and Scan Interpretation Summary

TCP flags are meaningful because they help Nmap interpret responses to different probes.

| Observation | Typical Interpretation |
|---|---|
| SYN-ACK following a SYN probe | Port is open |
| RST following a SYN probe | Port is closed |
| RST following an ACK probe | Port is unfiltered |
| No response to a UDP probe | Port may be open or filtered |
| No response to a FIN or NULL probe | May indicate an open port on conventional TCP implementations, but is not conclusive |

Interpretations depend on scan type, response context, operating system, and filtering behavior.

## 9. Practical Assessment Record

**Authorized lab:** [Lab name]

**Target:** [Authorized target identifier]

**Nmap version:** [Version]

**Host discovery:** [Actual observations]

**Scan type:** [SYN, Connect, UDP, or another technique]

**Ports discovered:** [Actual results]

**TCP responses observed:** [Actual results, where available]

**Services identified:** [Actual results]

**Post-scan enumeration:** [Commands and findings]

**Interpretation:** [Explain what the evidence indicates]

**Limitations:** [Describe inconclusive results or constraints]

**Lessons learned:** [Your own conclusions]

## References

- Nmap Reference Guide: https://nmap.org/book/man.html
- Nmap Port Scanning Techniques: https://nmap.org/book/man-port-scanning-techniques.html
- Nmap Service and Version Detection: https://nmap.org/book/man-version-detection.html
- Nmap Scripting Engine: https://nmap.org/book/nse.html
- TCP Specification (RFC 9293): https://www.rfc-editor.org/rfc/rfc9293.html

# SSH Honeypot Deployment & Attack Monitoring Using Cowrie

## 1. Project Overview

This project demonstrates the deployment and monitoring of an SSH honeypot using **Cowrie** on Ubuntu Server. Kali Linux was used as the testing client to connect to the honeypot and simulate SSH-based attacker interaction.

The objective was to observe SSH connection activity, capture authentication events, monitor interactive shell sessions, analyze connection metadata, and inspect SSH network traffic using Wireshark.

Cowrie generated structured logs that helped identify client IP addresses, usernames, authentication results, session durations, and connection termination events.

## 2. Project Objectives

- Deploy and configure an SSH honeypot using Cowrie.
- Identify exposed SSH services using Nmap.
- Simulate SSH connections from Kali Linux.
- Capture SSH authentication events.
- Monitor interactive shell sessions.
- Analyze Cowrie logs and JSON events.
- Inspect SSH communication using Wireshark.
- Understand how honeypots collect security telemetry.

## 3. Lab Environment

| Component | Configuration |
|---|---|
| Honeypot OS | Ubuntu Server |
| Honeypot Software | Cowrie |
| Testing OS | Kali Linux |
| Network Tool | Nmap |
| Packet Analyzer | Wireshark |
| Log Analysis | Cowrie logs and JSON |
| Virtualization | Oracle VirtualBox |
| Network Mode | VirtualBox network environment |
| Protocol | SSH |

### Network Configuration

| Device | IP Address |
|---|---|
| Ubuntu Cowrie Honeypot | `192.168.31.39` |
| Kali Linux Testing Machine | `192.168.31.188` |
| Honeypot SSH Port | `2222` |
| Kali Client Source Port | `37418` |

## 4. Cowrie Honeypot Deployment

Cowrie was deployed on the Ubuntu Server environment using a Python virtual environment.

The Cowrie service was started and its running status was verified.

### Service Verification

The following observations were confirmed:

- Cowrie service was running.
- Process ID observed: `7539`.
- SSH honeypot listening port: `2222`.
- Service process: `twistd`.

### Purpose

The honeypot provides a simulated SSH environment that records incoming connections and user interactions without exposing a real production shell.

## 5. Network Reconnaissance Using Nmap

Nmap was executed from Kali Linux to identify services running on the Ubuntu honeypot.

### Command

```bash
nmap -sV 192.168.31.39
```

### Observed Output

```text
Host: ubuntu-VirtualBox.lan
IP: 192.168.31.39

999 closed tcp ports (reset)

PORT     STATE SERVICE VERSION
2222/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u3
```

### Analysis

- Target host was reachable.
- TCP port `2222` was open.
- SSH service was detected on port `2222`.
- The default Nmap scan examined the top 1,000 TCP ports.
- The remaining 999 scanned ports were closed.

### Security Observation

Nmap successfully identified the SSH service exposed by the Cowrie honeypot.

## 6. SSH Connection Testing

An SSH connection was initiated from Kali Linux to the Cowrie honeypot.

### Command

```bash
ssh root@192.168.31.39 -p 2222
```

### Connection Details

| Parameter | Value |
|---|---|
| Client | Kali Linux |
| Client IP | `192.168.31.188` |
| Target | Ubuntu Cowrie |
| Target IP | `192.168.31.39` |
| Port | `2222` |
| Protocol | SSH |

The connection reached the SSH authentication prompt. A subsequent Cowrie log event confirmed successful simulated authentication and interactive shell access.

## 7. SSH Authentication Monitoring

Cowrie JSON logs were analyzed to identify authentication events.

### Observed Event

```json
{
  "eventid": "cowrie.login.success",
  "username": "root",
  "password": "[REDACTED]",
  "src_ip": "192.168.31.188",
  "session": "62d0b0d3994d",
  "protocol": "ssh",
  "timestamp": "2026-05-22T12:28:18.370015Z"
}
```

### Analysis

| Attribute | Observed Value |
|---|---|
| Event ID | `cowrie.login.success` |
| Source IP | `192.168.31.188` |
| Username | `root` |
| Authentication Result | Successful |
| Protocol | SSH |
| Session ID | `62d0b0d3994d` |
| Timestamp | `2026-05-22 12:28:18 UTC` |

### Finding

Cowrie recorded a successful authentication event for the root username from the Kali testing machine.

This confirms successful authentication within the simulated Cowrie environment, not compromise of the actual Ubuntu operating system.

## 8. Cowrie Session Log Analysis

Cowrie logs were monitored to understand SSH session activity.

### Session 1

| Attribute | Observed Value |
|---|---|
| Source IP | `192.168.31.188` |
| Username | `root` |
| TTY Session Duration | 284.6 seconds |
| Total Connection Duration | 330.4 seconds |
| Event | Session closed |
| Protocol | SSH |

### Session 2

| Attribute | Observed Value |
|---|---|
| Source IP | `192.168.31.188` |
| Username | `root` |
| TTY Session Duration | 180.1 seconds |
| Total Connection Duration | 195.6 seconds |
| Exit Status | `1` |
| Disconnect Reason | Disconnected by user |
| Logout Event | `avatar root logging out` |

### Analysis

Cowrie recorded session establishment, shell access, TTY activity, connection termination, and logout events.

The session durations demonstrate how honeypot logs can be used to investigate the length of SSH interactions.

## 9. JSON Log Analysis Using jq

Cowrie JSON logs were inspected using `jq` to understand structured security events.

### Command

```bash
tail -f var/log/cowrie/cowrie.json | jq .
```

### Event Types Observed

| Event ID | Purpose |
|---|---|
| `cowrie.session.connect` | Records a new SSH connection |
| `cowrie.client.version` | Records the SSH client version |
| `cowrie.client.kex` | Records key-exchange information and HASSH fingerprint |
| `cowrie.login.success` | Records successful authentication |
| `cowrie.client.size` | Records terminal dimensions |
| `cowrie.client.var` | Records client environment variables |
| `cowrie.session.params` | Records session parameters |
| `cowrie.session.closed` | Records connection termination and duration |

### Observed Connection Information

```text
Source IP:       192.168.31.188
Source Port:     37418
Destination IP:  192.168.31.39
Destination Port: 2222
Protocol:        SSH
Session ID:      62d0b0d3994d
```

### Security Observation

Structured JSON logs make it easier to extract, filter, and investigate authentication events and SSH session metadata.

## 10. Network Traffic Analysis Using Wireshark

Wireshark was used to inspect SSH communication between Kali Linux and the Cowrie honeypot.

### Display Filter

```text
ssh
```

### Observed SSH Packets

| Packet No. | Observation |
|---|---|
| 421 | Client SSH protocol identification |
| 423 | Server SSH protocol identification |
| 425 | Client Key Exchange Init |
| 427 | Elliptic Curve Diffie-Hellman Key Exchange Init |
| 428 | Server Key Exchange Reply |
| 429 | Client New Keys encrypted packet |
| 430 | Server encrypted packet |
| 431–434 | Encrypted SSH communication |
| 506–511 | Further encrypted SSH packets |

### SSH Client and Server Versions

```text
Client: OpenSSH_10.0p2 Debian-8
Server: OpenSSH_9.2p1 Debian-2+deb12u3
```

### Frame 434 Detailed Analysis

| Field | Value |
|---|---|
| Source IP | `192.168.31.39` |
| Destination IP | `192.168.31.188` |
| Source Port | `2222` |
| Destination Port | `37418` |
| Protocol | SSH |
| Frame Length | 146 bytes |
| TCP Payload | 80 bytes |

### Analysis

The packet capture showed the SSH protocol identification process, key exchange initialization, key exchange reply, and encrypted communication.

After the SSH key exchange, the packet payloads were encrypted. Therefore, application-level credentials and commands could not be directly read from these Wireshark packets.

## 11. HASSH Fingerprint Analysis

Cowrie recorded the SSH client's HASSH fingerprint.

```text
eeca2460550b9ded084ecf2f70a75356
```

The JSON event also recorded the client's supported key exchange, encryption, MAC, and compression algorithms.

### Security Importance

SSH client fingerprints can help security analysts identify and compare SSH client configuration patterns during network investigations.

## 12. Project Findings

The following findings were confirmed during the lab:

1. Ubuntu Cowrie honeypot was running and listening on TCP port `2222`.
2. Kali Linux successfully discovered the SSH service using Nmap.
3. SSH connections from `192.168.31.188` to `192.168.31.39` were recorded.
4. Cowrie recorded a successful simulated root authentication event.
5. Interactive shell activity and session termination were recorded.
6. Cowrie JSON logs captured SSH client version and HASSH fingerprint information.
7. Wireshark identified SSH protocol negotiation and encrypted packet exchange.
8. Cowrie recorded two session durations: 330.4 seconds and 195.6 seconds.

**Note:** The available evidence does not establish a failed-login count or a verified list of attacker-entered shell commands. These metrics are therefore not reported.

## 13. Tools and Technologies

- Ubuntu Server
- Kali Linux
- Cowrie SSH Honeypot
- Nmap
- Wireshark
- jq
- Oracle VirtualBox
- SSH

## 14. Conclusion

This project provided practical experience in deploying and monitoring an SSH honeypot using Cowrie.

The lab demonstrated network reconnaissance with Nmap, SSH interaction monitoring, authentication event collection, session log analysis, JSON log inspection, and SSH packet analysis using Wireshark.

The collected telemetry demonstrates how honeypots can support security monitoring, threat observation, and incident investigation by recording suspicious connection activity in a controlled environment.

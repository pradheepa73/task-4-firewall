# 🛡️ Task 4 — Firewall Configuration and Traffic Filtering with UFW

![Domain](https://img.shields.io/badge/Domain-Network_Security-red)
![Tool](https://img.shields.io/badge/Tool-UFW-blue?logo=ubuntu)
![Platform](https://img.shields.io/badge/Platform-Kali_Linux-557C94?logo=kalilinux)
![Status](https://img.shields.io/badge/Status-Completed-success)
![License](https://img.shields.io/badge/Use-Educational-lightgrey)

> A hands-on implementation of host-based firewall rules using **UFW (Uncomplicated Firewall)** on Kali Linux — demonstrating inbound traffic control, rule precedence, safe enablement practices, and rule rollback.

---

## 📌 Executive Summary

This report documents the configuration, testing, and rollback of firewall rules on a Kali Linux VM using UFW. A deny rule was applied to **TCP port 23 (Telnet)** — an obsolete and insecure protocol — to demonstrate traffic filtering. Connectivity to that port was tested and confirmed blocked. The rule was then removed to restore the original state, following the principle of **least disruption in a controlled environment**.

**Key Results:**
- UFW installed and enabled safely (SSH allowed *before* activation to prevent lockout)
- Inbound TCP/23 blocked and verified via connection test
- Inbound TCP/22 explicitly allowed for secure remote administration
- Test rule removed cleanly to restore baseline state

---

## 🎯 Objectives

- Configure host-based firewall rules on Linux using UFW.
- Demonstrate inbound traffic filtering on a specific port.
- Test rules to confirm real-world behavior.
- Follow safe enablement practices (no self-lockout).
- Restore the environment to its original state after testing.
- Document the process professionally for auditability.

---

## ⚖️ Scope & Ethics

| Item | Details |
|---|---|
| Scope | Local Kali Linux VM only |
| Authorization | Self-owned virtual lab |
| Test target | `localhost` (127.0.0.1) and Kali VM IP (`192.168.56.10`) |
| No external systems were scanned or affected | ✅ |

> [!IMPORTANT]
> All testing was performed on an isolated VirtualBox VM owned by the author. No production systems, public networks, or third-party hosts were touched.

---

## 🧰 Environment

| Component | Detail |
|---|---|
| Operating System | Kali Linux (VirtualBox VM) |
| Firewall Tool | UFW 0.36.x (iptables frontend) |
| Backend | `iptables` / `nftables` |
| Host Machine | Windows 10/11 |
| Network Range | `192.168.56.0/24` |

---

## 🔬 Methodology

```text
1. Verify UFW installation and initial status
2. Allow SSH (22/tcp) BEFORE enabling UFW (prevents lockout)
3. Enable UFW with --force
4. Block inbound Telnet (23/tcp) — deny rule
5. Test connectivity to port 23 to confirm block
6. Verify allowed SSH rule remains functional
7. Remove the test rule to restore original state
8. Document all commands and outcomes
```

---

## 💻 Commands Executed

### 1. Verify UFW is installed and check initial state
```bash
sudo ufw --version
sudo ufw status verbose
```

### 2. Allow SSH before enabling (safety step)
```bash
sudo ufw allow 22/tcp
```
> ⚠️ **Critical:** Enabling UFW without allowing SSH first can lock you out of remote machines. This is one of the most common firewall mistakes.

### 3. Enable UFW
```bash
sudo ufw --force enable
sudo ufw status verbose
```

### 4. Block Telnet (port 23)
```bash
sudo ufw deny 23/tcp
sudo ufw status numbered
```

### 5. Test the block
```bash
timeout 5 nc -zv localhost 23
```
**Result:** Connection timed out / refused → rule is working ✅

### 6. Remove the test rule (restore original state)
```bash
sudo ufw delete deny 23/tcp
sudo ufw status numbered
```

### 7. Verify final state
```bash
sudo ufw status verbose
```

---

## 📊 Firewall Rule Table

| # | Port | Protocol | Action | Purpose | State |
|---|---|---|---|---|---|
| 1 | 22 | TCP | **ALLOW** | Secure remote administration (SSH) | Permanent |
| 2 | 23 | TCP | **DENY** | Block insecure Telnet service | Temporary (removed after test) |
| — | — | — | Default Deny Inbound | UFW default policy | Active |

---

## 🧪 Test Results

| Test | Command | Expected | Actual |
|---|---|---|---|
| UFW enabled | `ufw status` | `Status: active` | ✅ Active |
| SSH allowed | `ufw status numbered` | Rule `22/tcp ALLOW` | ✅ Present |
| Telnet blocked | `nc -zv localhost 23` | Connection refused/timeout | ✅ Failed as expected |
| Rule removed | `ufw status numbered` | No 23/tcp rule | ✅ Clean |
| Final state | `ufw status verbose` | Only 22/tcp allowed | ✅ Verified |

---

## 🧠 How UFW Filters Traffic

UFW is a **frontend for `iptables`** (or `nftables` on modern kernels). When a packet arrives:

1. **Rule matching (top-down):** UFW evaluates rules in order. The **first match wins**.
2. **Stateful inspection:** UFW tracks connection state. Once a connection is `ESTABLISHED`, return traffic is automatically allowed — no explicit outbound rule required.
3. **Default policy:** If no rule matches, the default policy applies (typically `deny incoming`, `allow outgoing`).
4. **Logging:** UFW can log blocked packets to `/var/log/ufw.log` for auditing.

### Rule Flow Diagram
```
Incoming Packet
      │
      ▼
┌─────────────────────┐
│ Match rule #1 (22)  │──Yes──▶ ALLOW
└─────────────────────┘
      │ No
      ▼
┌─────────────────────┐
│ Match rule #2 (23)  │──Yes──▶ DENY
└─────────────────────┘
      │ No
      ▼
┌─────────────────────┐
│ Default: DENY       │
└─────────────────────┘
```

---

## 🎤 Interview Questions & Answers

### 1. What is a firewall?
A firewall is a network security system that monitors and controls incoming and outgoing traffic based on predefined rules. It acts as a barrier between trusted and untrusted networks.

### 2. Difference between stateful and stateless firewalls?
- **Stateful:** Tracks the state of active connections (NEW, ESTABLISHED, RELATED). Automatically allows return traffic. More secure and efficient.
- **Stateless:** Examines each packet in isolation against static rules. No memory of past connections. Faster but less intelligent.

### 3. What are inbound and outbound rules?
- **Inbound:** Control traffic entering the machine (e.g., allow SSH from anywhere).
- **Outbound:** Control traffic leaving the machine (e.g., block a malicious process from exfiltrating data).

### 4. How does UFW simplify firewall management?
UFW provides a human-readable interface on top of iptables' complex syntax. Instead of writing long `iptables` chains, you use simple commands like `ufw allow 22/tcp` or `ufw deny 23`.

### 5. Why block port 23 (Telnet)?
Telnet transmits all data — including usernames and passwords — in **plaintext**. Anyone sniffing the network can capture credentials. SSH (port 22) provides encrypted communication and is the modern replacement.

### 6. What are common firewall mistakes?
- Forgetting to allow SSH before enabling UFW → **remote lockout**
- Misordering rules (first match wins)
- Using `allow all` instead of `deny by default`
- Not logging traffic
- Allowing broad ranges (e.g., `0.0.0.0/0`) on sensitive ports

### 7. How does a firewall improve network security?
- Reduces attack surface by closing unnecessary ports
- Enforces access control policies
- Segments networks (DMZ, internal, external)
- Logs and alerts on suspicious traffic patterns
- Blocks known malicious IPs/regions

### 8. What is NAT in firewalls?
**Network Address Translation** rewrites source or destination IP addresses and ports as packets pass through the firewall. It's used to:
- Hide internal IPs from the internet (masquerading)
- Share a single public IP across many internal hosts
- Forward specific external ports to internal services (port forwarding)

---

## 🛡️ Remediation & Best Practices Applied

| Practice | Implementation |
|---|---|
| **Deny by default** | UFW defaults to deny inbound |
| **Allow only what's needed** | Only SSH (22) explicitly permitted |
| **Secure protocols only** | Telnet (23) blocked; SSH (22) preferred |
| **Safe enablement** | SSH allowed before UFW activation |
| **Restore state** | Temporary test rule removed after verification |
| **Documentation** | Every command and result captured in this report |

---

## 📁 Repository Structure

```text
task-4-firewall/
├── README.md                          # This report
├── screenshots/
     ├── 01-ufw-enabled.png             # UFW activation
     ├── 02-ufw-rules.png               # Rule list after enable
     ├── 03-block-telnet.png            # Deny rule added
     ├── 04-test-block.png              # Connection test failed
     ├── 05-allow-ssh.png               # SSH allow rule
     ├── 06-remove-block.png            # Test rule deleted
     └── 07-final-state.png             # Final firewall state
```

---

## 📸 Evidence

| Screenshot | Description |
|---|---|
| `01-ufw-enabled.png` | UFW activated with SSH pre-allowed |
| `02-ufw-rules.png` | Rule list showing 22/tcp ALLOW |
| `03-block-telnet.png` | Deny rule for 23/tcp applied |
| `04-test-block.png` | `nc` connection to port 23 timed out |
| `05-allow-ssh.png` | SSH rule verified present |
| `06-remove-block.png` | Test rule deleted successfully |
| `07-final-state.png` | Final clean firewall state |

---

## 🧾 Lessons Learned

1. **Order of operations matters.** Enabling a firewall before allowing SSH is a classic mistake that locks admins out of remote systems.
2. **UFW makes iptables usable.** Complex chains become one-liners — but understanding the underlying iptables behavior is still critical.
3. **Test your rules.** A rule in the config doesn't guarantee the expected behavior — always verify with a real connection attempt.
4. **Rollback matters.** In production, always know how to remove a rule (`ufw delete`) before adding one.
5. **Telnet should never be used.** Port 23 has no place in modern infrastructure; its plaintext nature makes it a guaranteed finding in any audit.

---

## 📚 References

- [UFW Official Documentation](https://help.ubuntu.com/community/UFW)
- [Ubuntu Firewall Guide](https://ubuntu.com/server/docs/security-firewall)
- [iptables Man Page](https://linux.die.net/man/8/iptables)
- [NIST SP 800-41 — Guidelines on Firewalls](https://csrc.nist.gov/publications/detail/sp/800-41/rev-1/final)
- [CIS Benchmarks for Linux](https://www.cisecurity.org/cis-benchmarks)

---

## ✅ Submission Checklist

- [x] UFW installed and verified
- [x] SSH allowed before enablement
- [x] Deny rule for port 23 applied
- [x] Block confirmed via connection test
- [x] Rule removed and state restored
- [x] All 7 screenshots captured
- [x] README documented
- [x] Repository published to GitHub

---

## 👤 Author

**Pradheepa M**  
B.Sc. Computer Science with Cybersecurity  
📧 pradheepa378@gmail.com  

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/pradheepa-m-051728372)
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=flat-square&logo=github&logoColor=white)](https://github.com/pradheepa73)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat-square&logo=gmail&logoColor=white)](mailto:pradheepa378@gmail.com)

---

<div align="center">

**"Empowering The Digital Defenders"**  
*Network Security Assessment — Task 4*

</div>

---

> **Prepared by:** Pradheepa.M  
> **Date:** 2026-10-06  
> **Repository:** [github.com/pradheepa73/task-4-firewall](https://github.com/pradheepa73/task-4-firewall)
```

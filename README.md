# virtual-network-attack-defense-lab
This project simulates and defends against ARP spoofing and ICMP flood (DoS) attacks in a virtualized lab using Kali and Ubuntu VMs. It includes Wireshark captures, arpwatch detection, static ARP hardening, and iptables firewall rules to mitigate attacks. A great hands-on showcase of Layer 2/3 network security.

# Simulation and Defense Against ARP Spoofing and Ping Flood Attacks

**Author**: Ifeoluwa (SysVenom)  
**Project Goal**: Simulate Layer 2 and Layer 3 attacks (ARP spoofing and ICMP flood) in a virtual lab and implement detection and mitigation strategies using Linux tools.

---

## 1. Introduction

This project simulates ARP spoofing and ICMP flood attacks in a controlled virtual environment using VirtualBox, Kali Linux, and Ubuntu. Tools like Wireshark, arpwatch, and iptables were used for detection and mitigation.

---

## 2. Virtual Lab Environment Setup

### 2.1 Tools and Software
- VirtualBox (Hypervisor)
- Kali Linux (Attacker) — `10.0.2.15`
- Ubuntu Desktop (Defender) — `10.0.2.4`

### 2.2 Network Configuration
- NAT networking
- Gateway IP: `10.0.2.1`
- Interfaces:
  - Ubuntu: `enp0s3`
  - Kali: `eth0`

### 2.3 Interface Verification
```bash
ip a
ip route
```
📌 *Insert Screenshot: Output of `ip a` and `ip route` for both VMs*

### 2.4 Traffic Simulation and Detection

#### 2.4.1 SYN Simulation
```bash
ping -c 4 10.0.2.4
ping -c 4 10.0.2.15
```
📌 *Insert Screenshot: Successful ping results from both machines*

#### 2.4.1.1 Snort Live Capture  
📌 *Insert Screenshot: Snort detection output (if used)*

#### 2.4.1.2 Wireshark Capture  
📌 *Insert Screenshot: Baseline traffic in Wireshark before attacks*

---

## 3. Attack Simulation

### 3.1 ARP Spoofing Attack

#### 3.1.1 Preparation on Kali
```bash
sudo apt update
sudo apt install dsniff
sudo arp-scan --interface eth0 --localnet
```

#### 3.1.2 Launch Attack
```bash
sudo arpspoof -i eth0 -t 10.0.2.4 10.0.2.1
sudo arpspoof -i eth0 -t 10.0.2.1 10.0.2.4
```

#### 3.1.3 Check ARP Table on Ubuntu
```bash
arp -n
```
**Before:**
```
10.0.2.1 -> 52:54:00:12:35:00
```

**After:**
```
10.0.2.1 -> 08:00:27:59:e2:dc
10.0.2.15 -> 08:00:27:59:e2:dc
```

📌 *Insert Screenshot: ARP table before and after spoofing*  
📌 *Insert Screenshot: Wireshark showing spoofed ARP replies*

---

### 3.2 Ping Flood (ICMP DoS Attack)

```bash
ping -f 10.0.2.4
sudo apt install hping3
sudo hping3 -1 --flood -p 80 10.0.2.4
```

📌 *Insert Screenshot: High CPU/network usage on Ubuntu*  
📌 *Insert Screenshot: Wireshark showing ICMP flood*

---

## 4. Defense Mechanisms

### 4.1 Detecting ARP Spoofing with arpwatch

#### 4.1.1 Installation and Execution
```bash
sudo apt update
sudo apt install arpwatch
sudo arpwatch -i enp0s3 -f /var/lib/arpwatch/arp.dat
```

#### 4.1.2 Monitor Logs
```bash
sudo tail -f /var/log/syslog
```
Example log:
```
arpwatch: changed ethernet address 10.0.2.1 08:00:27:59:e2:dc (52:54:00:12:35:00)
```

📌 *Insert Screenshot: arpwatch terminal logs showing detection*

---

### 4.2 Manually Setting Static ARP Entries
```bash
sudo ip neigh del 10.0.2.1 dev enp0s3
sudo ip neigh del 10.0.2.15 dev enp0s3
sudo arp -s 10.0.2.1 52:54:00:12:35:00
arp -n
```

📌 *Insert Screenshot: ARP table with static (PERM) entries*

---

### 4.3 Ping Flood Mitigation with iptables
```bash
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

#### Retest Results
- **Duration**: ~17.3 seconds
- **Packets Sent**: 1021
- **Packets Received**: 18
- **Packet Loss**: 98.24%
- **Latency**: Min 0.93 ms / Avg 2.05 ms / Max 8.66 ms

📌 *Insert Screenshot: hping3 or ping output showing high packet loss*

---

## 5. Challenges and Problems Faced

- Interface mismatch (Kali: `eth0`, Ubuntu: `enp0s3`)
- Initially empty ARP cache
- Manual cleanup of poisoned ARP entries
- arpwatch warning about missing `sendmail`
- Interpreting filtered packets in Wireshark

---

## 6. Key Learnings

- How to verify and interpret network interfaces
- ARP cache poisoning and protection
- Setting static ARP entries
- Real-time ARP monitoring with arpwatch
- Rate-limiting ICMP using iptables
- Analyzing traffic at packet level using Wireshark

---

## 7. Future Enhancements

- Automate static ARP setup on boot with `systemd`
- Deploy Snort/Suricata for IDS/IPS
- Centralized syslog server setup
- Scripting alert/block logic for MAC anomalies
- Simulate MITM, DNS spoofing, DHCP starvation
- Analyze SSL/TLS sessions for hijack attempts
- Automatic blocking based on arpwatch alerts

---

## 8. Conclusion

This project demonstrated the process of simulating and defending against ARP spoofing and ICMP flood attacks in a virtualized lab environment. Through tools like Wireshark, arpwatch, and iptables, I successfully monitored, captured, and blocked malicious traffic. The hands-on experience reinforced both offensive and defensive networking skills at the packet level.

📌 *Insert Screenshot: Final Wireshark capture showing filtered/fair traffic*

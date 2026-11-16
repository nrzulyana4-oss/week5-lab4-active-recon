# LAB 4: Active Reconnaissance & Footprinting

- **Name:** Siti Nurzulyana Binti Yusree
- **Class:** B04
- **Lecturer:** Ts. Adani Kamal
- **Target Machine:** DVWA (Damn Vulnerable Web Application) — pentest-ground.com (178.79.134.182)
- **Date:** 16 November 2026

---

## Introduction & Objective

This report documents an individual active reconnaissance and footprinting exercise carried out against an authorized, intentionally vulnerable target. Active reconnaissance differs from passive footprinting in that it interacts directly with the target system to map its network exposure and the services it is running, rather than relying only on publicly available information.

The objectives of this exercise were:
- To configure an attacker machine (Kali Linux) with network access and verify its own host identity.
- To identify the authorized target and enumerate its open TCP ports and service versions using Nmap.
- To interpret the collected evidence objectively — distinguishing what the scan results prove, what they suggest, and what limitations remain — without performing any exploitation or destructive testing.

---

## 1. Lab & Target Setup

The attacker machine was Kali Linux, running in Oracle VirtualBox and connected through a NAT adapter, which placed it on the `10.0.2.0/24` internal segment with outbound access to reach the authorized target. The target was the hosted Damn Vulnerable Web Application (DVWA) instance on Pentest-Ground, a public playground explicitly provided for security testing that permits scanning of its services.

| Item | Value |
|---|---|
| Attacker host | Kali Linux (10.0.2.15) |
| Attacker network / CIDR | 10.0.2.0/24 (gateway 10.0.2.2) |
| Target | DVWA on pentest-ground.com (178.79.134.182) |
| Target web port (DVWA) | 4280/tcp |
| Scope | Authorized active footprinting and service enumeration only. No exploitation, brute-forcing, or destructive testing. |

### Target Selection Justification

DVWA was selected because it is an intentionally vulnerable web application designed for education and security practice. The Pentest-Ground instance is a legally sanctioned target that openly allows scanning, making it a controlled and authorized environment for web-focused reconnaissance. Because this is a shared public instance, each application is periodically redeployed to a clean state and other users may interact with it at the same time.

![DVWA target application](img/dvwa-welcome.png)
*Figure 1: DVWA target application served on pentest-ground.com:4280*

---

## 2. Own Host Identification & Target Identification

Before probing the target, I verified the attacker machine's own network configuration using `ip -br addr` and `ip route`. This confirmed that the primary interface (`eth0`) was assigned `10.0.2.15` within the `10.0.2.0/24` subnet, with a default route via `10.0.2.2` providing outbound connectivity.

```
$ ip -br addr
lo    UNKNOWN  127.0.0.1/8 ::1/128
eth0  UP       10.0.2.15/24 ...

$ ip route
default via 10.0.2.2 dev eth0 proto dhcp src 10.0.2.15 metric 100
10.0.2.0/24 dev eth0 proto kernel scope link src 10.0.2.15 metric 100
```

The authorized target was then identified by name resolution: `pentest-ground.com` resolved to the public IP `178.79.134.182`, and the subsequent Nmap scan confirmed the host was up (0.24s latency). Because the target is a remote authorized host rather than a machine on the local subnet, identification was performed via DNS resolution and the service scan rather than a local ping sweep.

![Kali network configuration](img/attacker-host.png)
*Figure 2: Kali Linux network configuration (attacker host 10.0.2.15)*

---

## 3. Service Enumeration & Interpretation

A service and version detection scan was executed against the target using `nmap -sV -p 81,4280,5013,6379,7001,9000 pentest-ground.com` on 16 November 2026. The scan reported six open TCP ports:

| Port | State | Service | Version banner |
|---|---|---|---|
| 81/tcp | open | ssl/http | nginx 1.31.8 |
| 4280/tcp | open | ssl/http | nginx 1.31.8 (DVWA web application) |
| 5013/tcp | open | ssl/http | nginx 1.31.8 |
| 6379/tcp | open | redis | Redis key-value store 5.0.7 |
| 7001/tcp | open | ssl/http | Oracle WebLogic admin httpd |
| 9000/tcp | open | ssl/http | nginx 1.31.8 |

![Nmap service scan](img/service-scan.png)
*Figure 3: Nmap service and version detection results*

### Web Reconnaissance Scope

Port `4280` was identified as the primary focus of this lab because it serves the DVWA web application. The remaining ports were documented as part of the target's overall network footprint but were outside the web-reconnaissance scope.

### Analysis & Evidence Assessment

**What the evidence proves:** The host at `178.79.134.182` is reachable and actively listening on TCP ports 81, 4280, 5013, 6379, 7001 and 9000. Four of these (81, 4280, 5013, 9000) are web services fronted by an nginx 1.31.8 reverse proxy, one (6379) is a Redis 5.0.7 data store, and one (7001) is an Oracle WebLogic administration service.

**What the evidence suggests:** The web service on port 4280 corresponds to the DVWA application and aligns directly with the lab's web-reconnaissance scope. Because the web ports are fronted by nginx, the banner reflects the reverse proxy rather than DVWA's underlying PHP/Apache stack, which is hidden behind it. The exposed Redis and WebLogic services represent additional attack surface that would warrant further authorized investigation. However, no service is marked as confirmed vulnerable, because no active validation or exploitation was performed.

**Confidence:** High that the six ports are open and running the listed service types (direct scan response); Medium on exact application versions, since banners are reverse-proxy-derived and can be hidden or changed.

---

## 4. Limitations & Operational Risks

- **Limited port range:** only the six specified ports were scanned, not the full 65,535 TCP range or any UDP ports. Services on other ports could remain undetected (false negatives).
- **Reverse-proxy banners:** the web ports are fronted by nginx, so the detected versions describe the proxy, not the DVWA application behind it. Version-based conclusions about DVWA itself cannot be drawn from this alone.
- **Shared, redeploying target:** the instance is public and periodically reset to a clean state, and other users may be scanning it simultaneously, so results represent one point in time.
- **Detection footprint:** unlike passive footprinting, active scanning sends probe packets directly to the target and would be logged, and in a real engagement could trigger IDS/IPS alerts.
- **No validation or exploitation:** nothing is labelled "confirmed vulnerable" — only what the evidence indicates, with a stated confidence level.

---

## Conclusion

Through controlled active reconnaissance from the Kali Linux attacker host (`10.0.2.15`), the authorized DVWA target on pentest-ground.com (`178.79.134.182`) was identified and enumerated. Service and version detection confirmed six open TCP ports, with the DVWA web application exposed on port 4280 behind an nginx 1.31.8 reverse proxy, alongside a Redis data store and an Oracle WebLogic service.

While these service banners establish the target's active web and network footprint, the version indicators alone do not confirm exploitable vulnerabilities, both because the web stack is masked behind a reverse proxy and because no active validation was carried out. Overall, this exercise demonstrates how structured network discovery provides visibility into a system's exposure, while reinforcing the importance of evidence-based interpretation over premature conclusions.

---

## Reference

- Universiti Kuala Lumpur (UniKL). (2026). *Chapter 3 (Week 5): Reconnaissance & Footprinting — Analysis & Threat Intelligence* [Lecture slides].
- Pentest-Ground. (2026). *Vulnerable apps to benchmark your scanners and your skills.* https://pentest-ground.com

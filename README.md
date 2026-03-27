# 🔐 DNS Spoofing & MITM Detection (Wireshark)

## 📌 Overview
This project documents a cybersecurity investigation into a Man-in-the-Middle (MITM) attack involving ARP poisoning and DNS spoofing. The objective was to analyse captured network traffic and identify malicious behaviour using Wireshark.

---

## 🎯 Objectives
- Analyse DNS traffic using Wireshark
- Establish baseline (legitimate DNS behaviour)   
- Identify rogue DNS responses 
- Detect DNS spoofing attack
- Understand how MITM attacks redirect traffic  a  

---

## 🧰 Tools Used
- **Wireshark** 
- **Kali Linux**    

---

## 🌐 Network Overview

| Role | IP Address | Description |
|------|------------|-------------|
| Victim | 192.168.10.10 | Target user machine |
| Gateway | 192.168.10.1 | Network router |
| Attacker | 192.168.10.55 | Rogue system performing the MITM attack |
| DNS Server | 8.8.8.8 | Legitimate DNS resolver |

---

## 🔍 Step 1 – Baseline DNS Behaviour

**Filter Used:**
```bash
dns.flags.response == 1 && ip.src == 8.8.8.8
```

### Findings
- A DNS response was observed from **192.168.10.55**, which is not the legitimate DNS server  
- The rogue response resolved the domain to:
```text
corp-login.acme-corp.local → 192.168.10.55
```
## Analysis

- **192.168.10.55** is an internal host and not the legitimate DNS server  
- It is sending DNS responses, which normal client machines should not do  
- The same domain received conflicting responses from different sources  

This behaviour indicates **DNS spoofing**, where a malicious system forges DNS replies to redirect traffic.

## Attack Chain Identified

From analysing the traffic, it looks like this attack happened in multiple stages:

### 1. ARP Spoofing
The attacker first placed themselves between the victim and the network gateway.  
This allows them to see and control the traffic.

### 2. DNS Spoofing
When the victim tried to access `corp-login.acme-corp.local`, both the real DNS server and the attacker responded.

The attacker sent a fake response, pointing the domain to their own IP address instead of the real one.

### 3. Traffic Redirection
Because of this, the victim could be redirected to the attacker’s machine instead of the legitimate server.

This could be used to capture login details or display a fake website.

## 🧾 Conclusion

Based on the analysis, this appears to be a DNS spoofing attack as part of a Man-in-the-Middle scenario.

The attacker at **192.168.10.55** sent a forged DNS response for `corp-login.acme-corp.local`, redirecting the victim away from the legitimate server (**192.168.10.200**) to their own machine.

This shows how attackers can manipulate DNS responses to trick users into connecting to the wrong system, which could lead to credential theft or further compromise.

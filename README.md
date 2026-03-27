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

##  Network Overview

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
dns
```

### Findings
- I started by analysing all DNS traffic to understand what domains were being requested.
- From the “Info” column, I observed several domains including:

updates.acme-corp.local
corp-login.acme-corp.local
cdn.svc.net
www.google.com

## Step 2 - Legitmate DNS Behavior 
Filter Used: ```dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"```

### Findings
- This shows DNS responses from 8.8.8.8, the legitimate DNS server.
- The victim (192.168.10.10) is attempting to resolve the domain.
- The domain resolves to: corp-login.acme-corp.local → 192.168.10.200

### Analysis 
- This is unusual because normal client machines should not respond to DNS queries.
- The same domain is now resolving to a different IP address compared to the legitimate response.
- This indicates that 192.168.10.55 is acting as a rogue DNS server, sending forged responses.

## Attack Chain Identified
From analysing the traffic, the attack appears to happen in multiple stages:
### 1. ARP Spoofing
The attacker positions itself between the victim and the gateway
This allows interception and manipulation of traffic
### 2. DNS Spoofing
The victim requests corp-login.acme-corp.local
The legitimate DNS server responds with the correct IP
The attacker also sends a fake response pointing to itself
### 3. Traffic Redirection
The victim may connect to the attacker instead of the real server
This could lead to credential theft or fake login pages

<br>

---
## 📸 Screenshots
### Narrowing down DNS traffic

<br>

Filter: ```dns```
![image alt](https://github.com/sjmercene/sjmercene/blob/main/DNS.png)
- I used this filter to view all DNS traffic and identify domain names being requested on the network.
- From the “Info” column, I observed several domains including: updates.acme-corp.local, corp-login.acme-corp.local, cdn.svc.net, www.google.com
- Some domains such as www.google.com appear normal and expected.
- Internal domains like "updates.acme-corp.local" and "corp-login.acme-corp.local" are more interesting, as they may represent internal services.
- The domain corp-login.acme-corp.local stood out as a potential target, as login-related domains are commonly targeted in phishing or credential theft attacks.
- This domain was selected for further investigation in later steps.
  
<br>

### Legitimate DNS Response 
```bash
dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"
```
![image alt](https://github.com/sjmercene/sjmercene/blob/main/Legitimate%20DNS%20Response.png)


- This screenshot shows a DNS response from **8.8.8.8**, which is the legitimate DNS server.
- We can see that the victim machine (192.168.10.10) is attempting to resolve the domain corp-login.acme-corp.local.
- We also found out "192.168.10.55 → 192.168.10.10" which i have highlighted in this screenshot. 
This is an example of DNS spoofing. The attacker is telling the victim to visit the website it wants it to.  

### Rogue DNS only (Attacker View)

<br>

filter: ```dns.flags.response == 1 && ip.src != 8.8.8.8```

![image alt](https://github.com/sjmercene/sjmercene/blob/main/Rogue%20DNS.JPG)
- This filter was used to isolate DNS responses that do not originate from the legitimate DNS server (8.8.8.8).
- The results show DNS responses coming from internal systems, such as 192.168.10.55, which should not normally act as a DNS server.
- This indicates that a rogue system is attempting to impersonate a DNS server and send forged responses.
- This behaviour is consistent with a DNS spoofing attack, where the attacker redirects traffic by manipulating DNS responses.

<br>

## 🧾 Conclusion

Based on the analysis, this appears to be a DNS spoofing attack as part of a Man-in-the-Middle scenario.

The attacker at **192.168.10.55** sent a forged DNS response for `corp-login.acme-corp.local`, redirecting the victim away from the legitimate server (**192.168.10.200**) to their own machine.

This shows how attackers can manipulate DNS responses to trick users into connecting to the wrong system, which could lead to credential theft or further compromise.

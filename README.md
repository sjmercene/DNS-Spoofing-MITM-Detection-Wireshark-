# DNS Spoofing & MITM Detection (Wireshark )

## 📌 Overview
In this lab, I investigated a suspected Man-in-the-Middle attack involving ARP poisoning and DNS spoofing.

The packet capture (.pcap) file was sourced via a controlled lab environment on TryHackMe. After completing the lab, I decided to access the environment via VNC, exported the capture, and performed further independent analysis within my own Kali Linux VM using Wireshark.

I chose to do this beyond the guided exercise and explore the traffic in more depth, with the goal and curiosity of understanding how a real-world analyst would approach and investigate the attack.

Using Wireshark, I established a baseline of normal DNS behaviour and identified anomalies where DNS responses were being spoofed by an internal host rather than the legitimate external DNS server.

Instead of finding out what IP address was used by the attacker, I focused on how the attack occurred in the first place and how the DNS spoofing resulted in a victim accessing a fake site hosted in a malicious system.



**This project demonstrates my ability to:**

- Analyse DNS traffic in Wireshark
- Identify legitimate and suspicious DNS responses
- Compare expected vs unexpected network behaviour
- Detect signs of DNS spoofing
- Understand how MITM attacks can redirect victims
- Explain findings in an investigation-style format

---

## Objectives

- Review DNS traffic using Wireshark
- Identify normal DNS behaviour
- Locate DNS responses from the legitimate DNS server
- Detect rogue DNS responses from an internal attacker
- Determine whether DNS spoofing occurred
- Explain the attack chain clearly


---

## 🧰 Tools Used
- **Tool:** Wireshark
- **OS:** Kali Linux
- **Protocol Analysis:** DNS, ARP, TCP/IP
- **Scenario:** MITM / DNS Spoofing Investigation
   

---

##  Network Overview

| Role | IP Address | Description |
|------|------------|-------------|
| Victim | 192.168.10.10 | Target user machine |
| Gateway | 192.168.10.1 | Network router |
| Attacker | 192.168.10.55 | Rogue system performing the MITM attack |
| DNS Server | 8.8.8.8 | Legitimate DNS resolver |
| Real Server | 192.168.10.200 | Legitimate destination for corp-login.acme-corp.local |
---

## Investigation Summary
During the investigation, I focused on the domain:  

`corp-login.acme-corp.local`  

This domain stood out to me as login-related services are commonly targeted in credential harvesting and phishing attacks, which I became familiar with during my Certificate IV in Cyber Security studies.  

Using Wireshark, I analysed the DNS traffic to first understand what normal behaviour looked like, then compared it against anything unusual. I observed that the legitimate DNS server resolved this domain to:  

`192.168.10.200`  

However, I also identified DNS responses for the same query originating from:  

`192.168.10.55`  

Based on my understanding of how DNS should operate, this was immediately suspicious, as standard client machines should not generate DNS responses. Legitimate responses are expected to come from authorised DNS servers only and not internal.  

I also noticed multiple responses being returned for the same DNS query, which further indicated something was not right.  

From this information gathered, I concluded that an internal system was attempting to impersonate a DNS server, consistent with DNS spoofing techniques used to redirect victims to malicious destinations.  

---

## ✅ Verification  

### DNS Traffic Overview  

<img src="https://github.com/sjmercene/sjmercene/blob/DNS-MIDM-Images/DNS.png" width="900">  

**Filter Used:**  

```text
dns
```

I began by filtering for all DNS traffic to get a clear view of what domains were being requested across the network.  

From the Wireshark *Info* column, I observed several domain lookups, including:  

- `updates.acme-corp.local`  
- `corp-login.acme-corp.local`  
- `cdn.svc.net`  
- `www.google.com`  

Some domains, such as `www.google.com`, appeared normal and expected for typical user activity.  

However, internal domains like `updates.acme-corp.local` and `corp-login.acme-corp.local` drew more attention, as they likely represent internal services within the network.  

In particular, `corp-login.acme-corp.local` stood out to me the most. Based on my understanding of how attackers typically target authentication services, login-related domains are often used in credential harvesting attempts. For this reason, I prioritised further analysis on this domain to determine whether any suspicious behaviour was present.

---

## Legitimate DNS Response  

<img src="https://github.com/sjmercene/sjmercene/blob/DNS-MIDM-Images/Legitimate%20DNS%20Response.png" width="900">  

**Filter Used:**  
`dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"`  

I used this filter to narrow down the traffic to only DNS responses (not queries), specifically for the domain `corp-login.acme-corp.local`, and only from the known legitimate DNS resolver (`8.8.8.8`).  

My goal was to isolate a trusted response so I could clearly identify what the correct IP resolution should be before investigating any anomalies.  

From this, I confirmed that the victim machine (`192.168.10.10`) had issued a DNS query for the domain.  

The legitimate DNS response resolved the domain as follows:  

`corp-login.acme-corp.local → 192.168.10.200`  

This gave me a reliable baseline of expected DNS behaviour, which I later used to compare against any conflicting or suspicious responses.

---

## Rogue DNS Response Detected  

<img src="https://github.com/sjmercene/sjmercene/blob/DNS-MIDM-Images/Rogue%20DNS.JPG" width="900">  

**Filter Used:**  

`dns.flags.response == 1 && ip.src != 8.8.8.8`  

After establishing a baseline using the legitimate DNS server, I adjusted the filter to identify DNS responses that were not coming from the trusted resolver (`8.8.8.8`).  

The purpose of this was to quickly highlight any unexpected sources responding to DNS queries, which could indicate spoofing activity.  

From this, I identified DNS responses originating from:  

`192.168.10.55`  

Based on my earlier analysis, this was already suspicious because this IP address is not a legitimate DNS server within the network.  

Normally, client machines only send DNS queries but do not generate responses. From my observations the internal host returning DNS responses suggests that it was impersonating a DNS server.  

When compared with the legitimate response previously identified, this behaviour is consistent with DNS spoofing, where an attacker sends forged DNS replies to redirect a victim to a malicious destination.  

---

## 🛠️ Analysis & Problem Solving
**Suspicious DNS Behaviour**

**Problem:**
The victim received DNS responses from a system that was not the legitimate DNS server.

**Expected Behaviour:**
DNS responses should come from: `8.8.8.8`

**Observed Behaviour:**
A DNS response was also sent from: `192.168.10.55`

**Why This Is Suspicious:**
The attacker IP was not configured as the DNS server. Since it was still responding to DNS queries, this indicates that it was attempting to interfere with the victim’s DNS resolution.

**Result:**
This confirmed that 192.168.10.55 was behaving as a rogue DNS responder.

---

## 🛠️ Analysis & Problem Solving  

### Suspicious DNS Behaviour  

**Problem:**  
The victim machine received DNS responses from a system that was not the legitimate DNS server.  

**Expected Behaviour:**  
DNS responses should only originate from the configured resolver: `8.8.8.8`  

**Observed Behaviour:**  
In addition to the legitimate response, a DNS reply was also observed from: `192.168.10.55`  

**Why This Is Suspicious:**  
Based on standard DNS operation, client machines are expected to send queries, not responses. The IP address `192.168.10.55` was not configured as a DNS server, yet it was actively replying to DNS requests.  

This indicates that the host was attempting to interfere with the normal DNS resolution process by injecting its own responses.  

**Result:**  
This confirms that `192.168.10.55` was acting as a rogue DNS responder which implies with DNS spoofing behaviour.

---
## 🧾 Conclusion  

The investigation confirmed clear indicators of a DNS spoofing attack as part of a Man-in-the-Middle (MITM) scenario. The attacker (`192.168.10.55`) was observed sending rogue DNS responses to the victim machine (`192.168.10.10`).  

The targeted domain was:  
`corp-login.acme-corp.local`  

The legitimate DNS server correctly resolved this domain to:  
`192.168.10.200`  

However, conflicting responses from the attacker indicate an attempt to manipulate DNS resolution and redirect the victim to a potentially malicious destination.  

This behaviour is consistent with DNS spoofing techniques commonly used in MITM attacks to intercept or redirect user traffic.  

---

## 📚 What I Learned  

This project strengthened my ability to:  

- Analyse DNS traffic using Wireshark  
- Establish a baseline for legitimate network behaviour  
- Identify anomalous DNS responses from unauthorised sources  
- Understand why client machines should not generate DNS responses  
- Recognise how DNS spoofing is used within MITM attacks  
- Investigate how attackers can redirect victims to malicious systems  
- Clearly document and communicate packet-level findings  



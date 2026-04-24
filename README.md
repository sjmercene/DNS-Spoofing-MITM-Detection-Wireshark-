# DNS Spoofing & MITM Detection (Wireshark )

## 📌 Overview
In this lab, I investigated a suspected Man-in-the-Middle attack involving ARP poisoning and DNS spoofing.

Using Wireshark, I analysed captured network traffic to compare legitimate DNS behaviour against suspicious DNS responses coming from an unexpected internal host.

Rather than only identifying the final attacker IP, I worked through the traffic step by step to understand how the attack occurred and how DNS spoofing can redirect a victim to a malicious system.

This project demonstrates my ability to:

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

This stood out because login-related domains are high-value targets in phishing and credential theft attacks.

The legitimate DNS server resolved this domain to:

`192.168.10.200`

However, I also observed a DNS response from:

`192.168.10.55`

This was suspicious because normal client machines should not be responding to DNS queries. This indicated that the attacker was attempting to impersonate a DNS server and redirect the victim.

## ✅ Verification

### DNS Traffic Overview

<img src="https://github.com/sjmercene/sjmercene/blob/DNS-MIDM-Images/DNS.png" width="900">

**Filter Used:**

```text
dns
```

I started by filtering all DNS traffic to understand what domains were being requested on the network.

From the Wireshark Info column, I observed several domain lookups including:

updates.acme-corp.local
corp-login.acme-corp.local
cdn.svc.net
www.google.com

Some domains, such as www.google.com, appeared normal and expected.

However, internal domains such as updates.acme-corp.local and corp-login.acme-corp.local were more interesting because they may represent internal services.

The domain corp-login.acme-corp.local stood out because it appears to be related to authentication or login activity, making it a likely target for credential theft.

---

## Legitimate DNS Response
<img src="https://github.com/sjmercene/sjmercene/blob/DNS-MIDM-Images/Legitimate%20DNS%20Response.png" width="900">

Filter used:
`dns.flags.response == 1 && ip.src == 8.8.8.8 && dns.qry.name == "corp-login.acme-corp.local"`

I used this filter to identify DNS responses coming from the legitimate DNS resolver, 8.8.8.8.

This confirmed that the victim machine, 192.168.10.10, requested the domain: `corp-login.acme-corp.local`

The legitimate DNS response resolved the domain to:`corp-login.acme-corp.local -> 192.168.10.200` 

This gave me a baseline for what the correct DNS behaviour should look like.

---

## Rogue DNS Response Detected
<img src="https://github.com/sjmercene/sjmercene/blob/DNS-MIDM-Images/Rogue%20DNS.JPG" width="900">

Filter Used:

`dns.flags.response == 1 && ip.src != 8.8.8.8`

After confirming the legitimate DNS response, I filtered for DNS responses that did not come from the approved DNS server.

This revealed DNS responses coming from: `192.168.10.55`

This was suspicious because 192.168.10.55 is not the legitimate DNS server.

A normal client machine should not be replying to DNS queries. The fact that this internal host was sending DNS responses suggests that it was attempting to impersonate a DNS server.

This behaviour is consistent with DNS spoofing, where an attacker sends forged DNS responses to redirect the victim to a malicious destination.

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

## DNS Spoofing Confirmed

**Problem:**
The same domain was associated with different DNS responses.

**Legitimate Resolution:**

corp-login.acme-corp.local -> 192.168.10.200

**Suspicious Activity:**
The attacker sent a forged DNS response to the victim.

**Why This Matters:**
If the victim accepts the forged DNS response, the victim could be redirected away from the legitimate server. This could then allow the attacker to host a fake login page or capture sensitive information from the victim.

**Result:**
This confirms a DNS spoofing attempt targeting corp-login.acme-corp.local.

---
## Final Result

The investigation confirmed signs of a DNS spoofing attack as part of a Man-in-the-Middle scenario. The attacker at: `192.168.10.55`
sent suspicious DNS responses to the victim: `192.168.10.10`

The target domain was: `corp-login.acme-corp.local`

The legitimate server should have resolved to: `192.168.10.200`

This shows that the attacker attempted to manipulate DNS traffic and redirect the victim to a malicious destination.

**What I Learned**

This project helped me understand:

- How to investigate DNS traffic in Wireshark
- How to establish a baseline for legitimate DNS behaviour
- How to identify suspicious DNS responses
- Why client machines should not normally respond to DNS queries
- How DNS spoofing can be used in MITM attacks
- How attackers can redirect victims to malicious systems
- How to explain packet analysis findings clearly




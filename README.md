# 🔐 VNC Exploitation & Traffic Analysis

## 📌 Overview
This project documents a hands-on cybersecurity lab completed on TryHackMe.  
The objective was to identify exposed services, gain access to a target system, extract sensitive data, and analyse captured network traffic using Wireshark.

---

## 🎯 Objectives
- Perform network reconnaissance  
- Identify exposed services  
- Gain access to a remote system  
- Locate and extract sensitive files  
- Analyse network traffic for useful data  

---

## 🖥️ Environment
- **Attacker Machine:** Kali Linux (VirtualBox)  
- **VPN:** OpenVPN connection to TryHackMe lab network  
- **Target Machine:** Linux system with exposed VNC service  

---

## 🔍 Reconnaissance

### Nmap Scan
```bash
nmap -T4 <target-ip>
```
### Results

- 22/tcp → SSH  
- 80/tcp → Web (WebSockify)  
- 5901/tcp → VNC  

### Key Finding

Port 5901 indicated a VNC service, and port 80 revealed a WebSockify server, suggesting a web-based VNC interface.

## 📸 Screenshots

### Nmap Scan
![image alt](https://github.com/sjmercene/sjmercene/blob/69432a4149acb3a7d91e2cd8d1b3023c84cca732/NmapScan.jpg)
This scan identified three open ports on the target:
- 22/tcp (SSH)
- 80/tcp (WebSockify)
- 5901/tcp (VNC)

The presence of port 5901 indicated a VNC service, suggesting potential remote desktop access.


### VNC Access
![image alt](https://github.com/sjmercene/sjmercene/blob/b4407705d5e2610533ae5c84c7f7b458407de8f0/VNC%20Viewer.png)

The screenshot shows a successful connection to the target system via VNC.

A remote desktop session was established after identifying the VNC service on port 5901. The login was successful using weak credentials (`root`), providing full graphical access to the system.

This confirms that the VNC service was exposed and improperly secured, allowing unauthorized access.


### ARP Traffic Analysis
![image alt](https://github.com/sjmercene/sjmercene/blob/b4407705d5e2610533ae5c84c7f7b458407de8f0/ARP%20Traffic%20Overview.jpg)

The screenshot displays captured ARP packets in Wireshark.

The traffic shows standard ARP request and response behaviour:
- “Who has [IP address]?” → ARP request  
- “[IP address] is at [MAC address]” → ARP reply  

This indicates normal network communication where devices resolve IP addresses to MAC addresses.

No abnormal or suspicious ARP activity (such as duplicate replies or spoofing indicators) was observed during this capture.

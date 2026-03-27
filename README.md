🕵️‍♂️#4 SOC-Incident-Report-Traffic-Analysis-NEMOTODES
You work as a analyst at a Security Operation Center (SOC) for a medical research facility specializing in nemotodes. Alerts on traffic in your network indicate someone has been infected. You don't know which is more disgusting, the nemotodes or the malware.

🌎 BACKGROUND
A network intrusion was detected involving a Windows endpoint (10.11.26.183). The infection chain was triggered by a "drive-by download" or "web-inject" scenario. The user initially visited a legitimate but compromised website, classicgrand.com, which served as the entry point. This visit resulted in an unauthorized redirection to a malicious delivery domain, leading to the deployment of a Remote Access Trojan (RAT).

The program used to analyze the packets was Wireshark 🦈​

🌎​ Environment Details
- LAN segment range:  10.11.26[.]0/24 (10.11.26[.]0 through 10.11.26[.]255)
- Domain:  nemotodes[.]health
- Active Directory (AD) domain controller:  10.11.26[.]3 - NEMOTODES-DC
- AD environment name:  NEMOTODES
- LAN segment gateway:  10.11.26[.]1
- LAN segment broadcast address:  10.11.26[.]255

​🔎​ Findings:
- Infected IP Address: 10.11.26.183
- Infected MAC Address: d0:57:7b:ce:fc:8b
- Infected Hostname: DESKTOP-B8TQK49
- Infected Windows User Account Name: oboomwald

⚠️ IOCs:
- Malicious IP Addresses: 194.180.191.64 (Established C2 channel via HTTP/HTTPS over port 443) (NetSupport RAT)
- Suspicious IP Addresses: 104.117.247.184 (It appears the attacker was trying to geolocate the victim.)
- Malicious Domains: modandcrackedapk.com (He was redirected here) & classicgrand.com (Onset of infection)
- Used port: 443 (TCP) (Used for RAT traffic)
- Dangerous Protocol Detected: SMBv1

🛜 Network Behavior:
Initial Redirect: Traffic analysis shows a referral from classicgrand.com to modandcrackedapk.com. 
C2 Establishment: The infected host established a persistent connection with the IP 194.180.191.64 via TCP Port 443. Although using a port typically reserved for HTTPS, the traffic was identified as NetSupport RAT protocol activity (Check-ins and Remote Admin responses). 
Reconnaissance: The malware performed a GeoLocation Lookup via ip-api.com (104.26.1.231) to identify the victim's physical location—a common tactic for botnet categorization. 
Internal Vulnerability: Detection of SMBv1 and NetBIOS overflow attempts suggests the malware or the attacker may have attempted lateral movement or credential harvesting within the local network.

🧐​ Process & Filters:
As a SOC analyst, we have these alerts:

<img width="1600" height="637" alt="image" src="https://github.com/user-attachments/assets/c0fa310d-f9c8-4332-83bb-e319c34c4a38" />

These alerts allow us to identify certain potentially useful data. We have both external and internal IP addresses. They indicate the use of the SMBv1 protocol (unsafe), the presence of dangerous domains such as "modandcrackedapk.com", port 443 (which is being used for a potentially malicious purpose), activity from NetSupport RAT (Trojan), and finally, an attempted geolocation.

<img width="952" height="569" alt="image" src="https://github.com/user-attachments/assets/06b7d2a6-7757-42fd-a5e2-f2174cb47250" />

ip.addr == 10.11.26.183 && nbns
- With this filter we know Infected Hostname, Infected IP address and infected MAC Address

<img width="772" height="719" alt="image" src="https://github.com/user-attachments/assets/48da7698-4f64-4842-b79f-eb00c37ca068" />

ip.addr == 10.11.26.183 && kerberos.CNameString
- With this filter we know Infected Windows User Account Name

<img width="1630" height="363" alt="image" src="https://github.com/user-attachments/assets/f5d34eab-604f-411e-9c9f-121c9c34a193" />

By using beaconing and ip.addr == 10.11.26.183 && dns, we can discover a malicious domain.

😎 Conclusion:
The endpoint has been successfully compromised and recruited into a Botnet infrastructure controlled via the NetSupport RAT. This tool provides the attacker with full remote desktop capabilities, file system access, and the ability to deploy secondary payloads such as ransomware or info-stealers. The activity is classified as a Critical Security Incident due to the established persistence and the high level of control held by the external actor.

🛡️ Recommendations as a Cybersecurity Professional:
1) Isolate the Host: Immediately disconnect 10.11.26.183 from the network to prevent further data exfiltration or lateral movemen
2) Credential Reset: Force a password change for the user account oboomwald and any other accounts used on that machine, as they should be considered compromised.
3) Eradication: Perform a full wipe and re-image of the affected workstation. Hand-cleaning a RAT infection is unreliable due to potential hidden persistence mechanisms.
4) Network Hardening: * Disable SMBv1 across the entire domain to mitigate legacy exploit risks. Block the identified IOC IPs (194.180.191.64, 104.117.247.184) at the corporate firewall.
5) Web Filtering: Implement an EDR or Web Proxy solution to block access to known malicious domains like modandcrackedapk.com.

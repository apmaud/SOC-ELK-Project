# SOC-ELK-Project

## Objective

The SOC Lab project was designed to build a functional Security Operations Center (SOC) environment using real-world security tools and multiple interconnected machines. The lab focused on the deployment and configuration of Elastic and Kibana for log ingestion, monitoring, and alerting, as well as integrating alerts with the osTicket ticketing system to support incident tracking and analyst workflows. 
An attacker system using the Mythic framework was introduced to generate realistic security events, allowing alerts to be investigated and escalated by a simulated SOC analyst. This project provided hands-on experience with a production-style SOC stack and reinforced practical understanding of threat detection, alert handling, and the end-to-end threat analysis process.

### Skills Learned
- Advanced understanding of SOC architecture and workflows using a multi-system, real-world security stack
- Proficiency in deploying, configuring, and operating the Elastic Stack (Elastic and Kibana) for log ingestion, alerting, and threat monitoring
- Ability to integrate SIEM alerts with ticketing systems such as osTicket to support incident tracking and analyst workflows
- Enhanced knowledge of cloud computing concepts, including virtual private cloud (VPC) networking, segmented server deployment, and secure inter-service communication
- Experience generating and analyzing attack telemetry using adversary systems to simulate real-world threat scenarios
- Strengthened problem-solving and analytical skills through troubleshooting distributed systems, alert pipelines, and cloud-based service issues

### Tools Used
- Elastic: SIEM and log ingestion
- Elasticsearch: Logstash pipeline collecting telemetry from multiple sources and "stashing" it
- Elastic Defend: EDR
- Kibana: Log visualization, dashboards, and alerting
- osTicket: Incident tracking and case management
- Kali Linux: Attacker simulation and threat generation
- Mythic: C2 (Command and Control) framework
- Vultr: Cloud hosting and VPC-based server infrastructure

## Steps

### 1. Created plan for overall structure of the project

<img width="705" height="801" alt="Mini-SOC-Project drawio" src="https://github.com/user-attachments/assets/8d4782c2-b80a-45c7-a14d-8c5b82760134" />

### 2. Servers set up on Vultr and environment configurations
<img width="1864" height="966" alt="1 serverlist" src="https://github.com/user-attachments/assets/79034786-97ac-426a-a78c-6f669ec566ae" />
<img width="1866" height="964" alt="2- VPC" src="https://github.com/user-attachments/assets/8abf69a4-999f-49a6-b36f-7d741059b3e1" />
<img width="1864" height="963" alt="3 - firewall policies" src="https://github.com/user-attachments/assets/5c501e7f-55aa-4666-a1e7-5bc837ad1dd3" />
<img width="1869" height="959" alt="4- firewall policies" src="https://github.com/user-attachments/assets/1243f567-ad3d-46ac-b06f-9fdd2a8af151" />
<img width="1866" height="957" alt="5" src="https://github.com/user-attachments/assets/5d2f6445-5c0b-490a-bdfe-d94e75f66f73" />

All servers for each component of the lab system were spun up and each component was installed. All necessary servers were added to a VPC and firewall policies were optimized to allow access to these servers from my specific host IP and for communication in between the servers themselves.
- APMAUD-ELK: Elastic was installed. Then Kibana was enrolled with that Elasticsearch instance's token. Kibana encryption keys were stored and port settings on Vultr and server were optimized.
- APMAUD-FLEET: Fleet server installed from Elastic Kibana dashboard and linked.
- APMAUD-Windows-Server: Sysmon installed and Elastic agent installed from dashboard, to its specific agent policy. It collects Defender and Sysmon logs in a customizable fashion.
- APMAUD-Linux-Server: Elastic agent installed for its specific agent policy. It forwards the authentication logs.
- APMAUD-osTicket: osTicket set up per documentation on Windows Server. All necessary dependencies like Apache set up and ports 80 and 443 enabled. Database was created inside the osTicket dashboard, and permissions given to the root user for access.
- APMAUD-MYTHIC: Mythic framework installed on the Ubuntu server. Specifics below in the lab write-up!

### 3. The attack process overview
<img width="501" height="1246" alt="Attack-Diagram drawio" src="https://github.com/user-attachments/assets/ef12ffd2-1968-4d8b-a5aa-ee5b57fa5304" />

This will generate our telemetry and further our understanding of:  The C2 process, sifting through logs in Elastic and Kibana, creating rules/alerts, creating dashboards with the data, setting up an EDR using Elastic Defend and linking alerts to our ticketing system (osTicket).


### 4. Brute forcing into the target Windows machine

<img width="1920" height="1080" alt="1-wordlist setupo kali" src="https://github.com/user-attachments/assets/a87db811-9a58-4590-808f-e428d02705bf" />
<img width="1276" height="1010" alt="2-bruteforce and RDP login" src="https://github.com/user-attachments/assets/bcdcde31-e130-43b3-ab74-57846b8e32f9" />

In a word-list for possible passwords to try, the correct password was put in just to give us a proof of concept. Hydra was ran to bruteforce an RDP connection to the target machine using that list.

<img width="1278" height="1007" alt="3-powershell info" src="https://github.com/user-attachments/assets/02ae1344-44cf-4849-9005-c5961d9e8bec" />
<img width="1263" height="992" alt="4-windows-def" src="https://github.com/user-attachments/assets/b353685b-16d8-46c6-a004-ed77afc9220e" />


After RDPing from the attack Kali machine into the target Windows machine, target system info and Windows Defender can be disabled.


### 5. Mythic server payload was set up in order to infect the target machine
<img width="1916" height="957" alt="5 payload set up on mythic using apollo and http c2 profile" src="https://github.com/user-attachments/assets/ed606adc-d1bf-4a86-9a3a-dc3f1d2f8149" />
<img width="1274" height="704" alt="6- putting payload on mythic server, and hosting on port 9999 to windows machine from kali-rdp" src="https://github.com/user-attachments/assets/110696ec-1bbe-4b5c-9734-05b8d884cfbc" />

Downloaded specific agent Apollo that is compatible with Windows and installed the Mythic http C2 Profile. Downloaded the agent onto the mythic server itself, renamed it to either svchost-apmaud.exe or apmaud-30.exe and hosted a python3 http server port 9999 to allow downloads through the port from the Mythic server's public IP.

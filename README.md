# SOC-ELK-Project

## Objective

The SOC Lab project was designed to build a functional Security Operations Center (SOC) environment using real-world security tools and multiple interconnected machines. The lab focused on the deployment and configuration of Elastic and Kibana for log ingestion, monitoring, and alerting, as well as integrating alerts with the osTicket ticketing system to support incident tracking and analyst workflows. 
An attacker system using the Mythic framework was introduced to generate realistic security events, allowing alerts to be investigated and escalated by a simulated SOC analyst. This project provided hands-on experience with a production-style SOC stack and reinforced practical understanding of threat detection, alert handling, and the end-to-end threat analysis process.

**NOTE** ALL server information and instances used in the project were destroyed... so don't you try and use one of my IPs 😃 They were left in the clear on some of the following pictures on purpose!    
**NOTE** All red-teaming tools and processes in this lab were used in an environment that I own and have permission to exploit!

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
***

### 1. Created plan for overall structure of the project

<img width="705" height="801" alt="Mini-SOC-Project drawio" src="https://github.com/user-attachments/assets/8d4782c2-b80a-45c7-a14d-8c5b82760134" />

***

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

***

### 3. The attack process overview
<img width="501" height="1246" alt="Attack-Diagram drawio" src="https://github.com/user-attachments/assets/ef12ffd2-1968-4d8b-a5aa-ee5b57fa5304" />

This will generate our telemetry and further our understanding of:  The C2 process, sifting through logs in Elastic and Kibana, creating rules/alerts, creating dashboards with the data, setting up an EDR using Elastic Defend and linking alerts to our ticketing system (osTicket).

***

### 4. Brute forcing into the target Windows machine

<img width="1920" height="1080" alt="1-wordlist setupo kali" src="https://github.com/user-attachments/assets/a87db811-9a58-4590-808f-e428d02705bf" />
<img width="1276" height="1010" alt="2-bruteforce and RDP login" src="https://github.com/user-attachments/assets/bcdcde31-e130-43b3-ab74-57846b8e32f9" />

In a word-list for possible passwords to try, the correct password was put in just to give us a proof of concept. Hydra was ran to bruteforce an RDP connection to the target machine using that list.

<img width="1278" height="1007" alt="3-powershell info" src="https://github.com/user-attachments/assets/02ae1344-44cf-4849-9005-c5961d9e8bec" />
<img width="1263" height="992" alt="4-windows-def" src="https://github.com/user-attachments/assets/b353685b-16d8-46c6-a004-ed77afc9220e" />


After RDPing from the attack Kali machine into the target Windows machine, target system info and Windows Defender can be disabled.

***

### 5. Mythic server payload was set up in order to infect the target machine
<img width="1916" height="957" alt="5 payload set up on mythic using apollo and http c2 profile" src="https://github.com/user-attachments/assets/ed606adc-d1bf-4a86-9a3a-dc3f1d2f8149" />
<img width="1274" height="704" alt="6- putting payload on mythic server, and hosting on port 9999 to windows machine from kali-rdp" src="https://github.com/user-attachments/assets/110696ec-1bbe-4b5c-9734-05b8d884cfbc" />

Downloaded specific agent Apollo that is compatible with Windows and installed the Mythic http C2 Profile. Downloaded the agent onto the mythic server itself, renamed it to either svchost-apmaud.exe or apmaud-30.exe and hosted a python3 http server port 9999 to allow downloads through the port from the Mythic server's public IP.

<img width="1263" height="995" alt="7- established and can see mythic server staarting with 107  on there" src="https://github.com/user-attachments/assets/71c71046-9199-417c-83c2-957a6d764007" />

The Windows machine is infected once the svchost-apmaud.exe file is ran! We can see the Mythic server's IP connecting here.


<img width="1917" height="958" alt="8 callback commands" src="https://github.com/user-attachments/assets/5b744930-ff8c-4afb-89c6-e9e1e4bdfda1" />
<img width="1917" height="957" alt="9-downloaded passwords txt" src="https://github.com/user-attachments/assets/73f8bec0-702d-4a42-a11b-25d76843fa3b" />

Now that the Mythic agent Apollo has infected the Windows machine, the mythic server can use callbacks to extract information and even download our target passwords.txt file!

***

### 6. Detecting and visualization of traffic on the SIEM
<img width="1918" height="956" alt="10-new rule to detect apollo agent" src="https://github.com/user-attachments/assets/a8e36220-b9a4-44cd-9953-4de6786377ac" />
<img width="1865" height="959" alt="10 5 rules" src="https://github.com/user-attachments/assets/44c797cd-7826-48f7-b9b7-0709ea4170ef" />

Rules to detect malicious traffic were added to Elastic, using specific search queries such as event IDs and codes. Here we can specifically see the rule for the Apollo agent. Each rule will alert when activated.


<img width="1863" height="959" alt="11" src="https://github.com/user-attachments/assets/0c7dbd80-5d99-479e-b3a1-5fb0685e8fb0" />
<img width="1918" height="956" alt="11-dashboard for sus activity" src="https://github.com/user-attachments/assets/e0bb0d60-de94-4b03-a331-e17387fbc8ea" />

Dashboards were made in Elastic Kibana to provide easy visualization of suspiscious activity based on the rules and alerts.

***

### 7. Our ticketing system, osTicket was connected to Elastic
<img width="1866" height="960" alt="13-webhook" src="https://github.com/user-attachments/assets/d0cae0d2-b037-4c17-89b1-5a43429bd722" />
<img width="1865" height="960" alt="13 1 OS-ticket" src="https://github.com/user-attachments/assets/a0d7d739-627f-4c7b-b820-8e7b14d0d98b" />

OS Ticket was installed and connected via Webhook on Elastic.

<img width="1865" height="965" alt="15 rule to ticket" src="https://github.com/user-attachments/assets/915f5b1a-cb1a-43fe-97ba-62d660ec9c92" />


On each rule, edit the Action to be taken to submit a ticket via that Webhook with the appropriate information and fields.

<img width="1865" height="960" alt="16-mythic alert generated from rule" src="https://github.com/user-attachments/assets/75d824a7-3d4a-470e-bd33-1612867aa768" />
<img width="1862" height="955" alt="17- ticket generated from mythic alert" src="https://github.com/user-attachments/assets/d81ebc38-07ff-4a80-bba6-43391fc80c31" />

Here we can see an example of an alert from the Mythic Apollo threat come through on Elastic and be routed to the osTicket for assignment to a SOC operator.

***

### 8. Endpoint Detection and Response

<img width="1866" height="964" alt="18- EDR on windows server" src="https://github.com/user-attachments/assets/ae9827ac-d6b0-4ffc-8726-02bdaee7a56b" />

Elastic defend was downloaded onto the Windows Server from the Kibana dashboard directly! This is possible because of our handy fleet server and policy!

<img width="1262" height="720" alt="19- elastic defend blocked" src="https://github.com/user-attachments/assets/df2db5f8-da90-4c9c-a676-3d28669183c0" />

When running the Mythic agent executable again on the target Windows machine, Elastic Defend now detects it and blocks it from running!

<img width="1864" height="960" alt="20 - malware prevention alert" src="https://github.com/user-attachments/assets/9a532ca4-d49e-4f00-8ad9-9a1ba1b95559" />

This reflects in Elastic as a Malware Prevention Alert and as a new rule!

<img width="1867" height="966" alt="21 isolation from elastic defend detection" src="https://github.com/user-attachments/assets/5952fa84-d8e1-44cc-ab7c-8662de45c938" />
<img width="1267" height="714" alt="22 isolated" src="https://github.com/user-attachments/assets/7d37d285-fe70-47ea-a6d4-48221d573f0e" />

An action can be added to this new rule, particularly the isolation action might be a good idea! It isolates the machine from the rest of the network and can only be removed within the dashboard itself. Normally the user would contact their Network admin to get this done or taken care of


***

That's it! If you made it this far, thanks for sticking around. I learned a great deal about the entire SOC process from this lab and I hope I was able to convey this!

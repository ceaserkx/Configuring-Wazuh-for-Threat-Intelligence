# Threat Intelligence Analysis with Wazuh

Wazuh is an open-source SIEM and XDR platform that can be deployed for free on local or cloud environments. I particularly love Wazuh for its silk-smooth GUI and simple functionality. For this project I utilized Wazuh to conduct Threat Intelligence on a collection
of endpoints after generating attack traffic with various red team tools.

## Contents
- [Scope](#scope)
- [Architecture](#architecture)
- [Componenets](#components)
- [Procedure](#procedure)

## Scope
In this project, I configured a controlled environment with five Ubuntu virtual machines on the Linode cloud platform, with one hosting the Wazuh console and four acting as vulnerable endpoints. The vulnerable machines had their firewalls set to allow all inbound traffic
(0.0.0.0/0) and were grouped within the same subnet for simplified management. In a real situation this kind of configuration would be cautioned against, but this just serves as a means to generate malicious traffic. I deployed Wazuh SIEM + XDR with a web GUI and installed 
agents on each endpoint. To test security measures, I simulated intrusion attempts using Kali Linux, employing [Hydra](https://github.com/vanhauser-thc/thc-hydra) for SSH brute-forcing, and [Test My NIDS](https://github.com/3CORESec/testmynids.org) for attacks like malware injection. Leveraging the MITRE ATT&CK 
framework, I analyzed security events to identify Tactics, Techniques, and Procedures, including Credential Access and Lateral Movement, while also reviewing critical CVEs affecting the endpoints. This project serves as a learning tool to better understand how 
SIEMs retrieve and aggregate traffic.

## Architecture 

(!) 

## Components 
### Infrastructure 
- Linode Cloud Platform
- Client Systems (Endpoints) - (5) Ubunutu, (1) Kali
- Firewall - Linode Cloud Firewall
### Security Tools
- SIEM - Wazuh
- SSH Bruteforcing - [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- Various attacks types, i.e. Malware injection - [Test My NIDS](https://github.com/3CORESec/testmynids.org)
## Procedure
(!)

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

<a href="https://imgur.com/r0rtjKS"><img src="https://i.imgur.com/r0rtjKS.png" title="source: imgur.com" /></a>

## Components 
### Infrastructure 
- Linode Cloud Platform
- Client Systems (Endpoints) - (5) Ubunutu, (1) Kali
- Firewall - Linode Cloud Firewall
### Security Tools
- SIEM - Wazuh
- SSH Bruteforcing - [Hydra](https://github.com/vanhauser-thc/thc-hydra)
- Various attacks types, i.e. Malware injection - [Test My NIDS](https://github.com/3CORESec/testmynids.org) by 3CORESec
## Procedure

Procedure below is not granularly step-by-step. This is to get a quick snapshot of steps taken to execute this project.

### 1. Linode Network Configuration
#### 1.a Configure Wazuh Linode
  - Select settings such as machine region, OS etc.
  - Create Firewall "ManagerWall", deny all traffic except ICMP traffic for troubleshooting via ping command, apply firewall to Wazuh Linode only
  - Should not have to allow the ips of the Ubuntu target endpoints as installing the Wazuh agent on those endpoints will allow communication between them and the Wazuh Console
  - After deployment, SSH into the Linode and execute the following
    
    ```
    curl -sO https://packages.wazuh.com/4.9/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
    ```
  - After installation is complete, create and store wazuh credentials in a safe space. Navigate to "https://YOUR-WAZUH-DASHBOARD-IP>" and login.
#### 1.b Configure Ubuntu insecure endpoints
- Repeat steps for 1.a with the exception of the Firewall
- Create Firewall "VulnFirewall", allow all traffic inbound and outbound
### 2. Deploy Wazuh Agents
#### 2.a Deploy Wazuh Agents from Wazuh Console
- Login to Wazuh Dashboard
- Navigate to https://YOUR-WAZUH-DASHBOARD-IP/app/endpoints-summary#/agents-preview/
- Follow installation guide, ensure following command is run on the respective ubunutu endpoint (if amd64 architectuure)
  
```
wget https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.9.1-1_amd64.deb && sudo WAZUH_MANAGER='(local-ip-of-the-endpoint)' dpkg -i ./wazuh-agent_4.9.1-1_amd64.deb
```
- On the ubuntu endpoint ensure following command is ran
  
  ```
  sudo systemctl daemon-reload
  sudo systemctl enable wazuh-agent
  sudo systemctl start wazuh-agent
  ```
### 3. Now the Fun Part; Generate Alerts
#### 3.a Using Hydra in Kali for SSH brute forcing
- Launch Kali Linux and navigate to Terminal
- We will select an ubuntu target at random; do some recon just for visibility, run:
  ```
  sudo nmap -sV -O (ip-address of machine; local ip if kali installed w/ linode cluster; public ip if kali installed on VM or local machine)
  ```
  - sV retrieves target's version for the respective port. i.e. Port 80, Apached httpd 2.4.57
  - O attempts to retrieve target OS
- For the linode machines, SSH service should be open and running, so we can attempt a bruteforce. (Althought port 22 is a common and expected port to open, this is why proper port management is important).
- Attempt an SSH bruteforce from a wordlist, run:
  ```
  hydra -l (username) -P /usr/share/wordlists/wfuzz/others/common_pass.txt ssh://(linode ip you are targeting)
- -l will login with LOGIN name, or load several logins from FILE
- -p will try password PASS, or load several passwords from FILE
- hydra will attempt the bruteforce with the wordlists, and this should generate events in the Wazuh Console (will navigate to these events in a moment)
#### 3.b Using Test My NIDS 
- Test My NIDS by 3CORESec is a quick way to generate alerts for Wazuh
- Navigate to the same (or a different machine if you want to test multiple endpoints) ubuntu machine and run,
```
curl -sSL https://raw.githubusercontent.com/3CORESec/testmynids.org/master/tmNIDS -o /tmp/tmNIDS && chmod +x /tmp/tmNIDS && /tmp/tmNIDS
```
- A menu should appear with a few options, I like option 11

<a href="https://github.com/3CORESec/testmynids.org/raw/master/assets/imgs/screenshot.png"><img src="https://github.com/3CORESec/testmynids.org/raw/master/assets/imgs/screenshot.png" title="source: imgur.com" /></a>

- This should be enough to generate attack traffic in Wazuh
#### 3.c Analyze Malicious Traffic and TTPs indicated by Wazuh
- In the Wazuh Console, navigate to >**Threat Hunting** >**Events**
- Several logs should have been generated with _rule.descriptions_ such as _"sshd: authentication failed"_ or _"sshd: Attempt to login using a non-existent user"_
- Navigate to **hourglass icon** on the far left.
- Analyzing the log event, we can see items such as _data.scrip_, which is where the particular attack originated. Wazuh will also mention related mitre techniques associated with the event with _rule.mitre.technique_.
  <a href="https://imgur.com/nXbLALM"><img src="https://i.imgur.com/nXbLALM.png" title="source: imgur.com" /></a>
- Navigate to the particular target endpoint
- We can utilize the given dashboards to create a quick analysis of all of the attacks this particular endpoint has undergone. Furthermore, if we navigate to **MITRE ATT&CK** tab near the top, we have additional metrics on the overall attack patterns for this endpoint.
  <a href="https://imgur.com/tq2rzHo"><img src="https://i.imgur.com/tq2rzHo.png" title="source: imgur.com" /></a>
<a href="https://imgur.com/S0KxEkY"><img src="https://i.imgur.com/S0KxEkY.png" title="source: imgur.com" /></a>
- A very valuable piece of threat intelligence we can analyze is if we navigate to the **Vulnerability Detection** tab. This page outlines the top CVEs we should address for this particular endpoint. In a real-life scenario, we could generate a report and provide patching instructions for the respective security team.
  <a href="https://imgur.com/QxxMlMb"><img src="https://i.imgur.com/QxxMlMb.png" title="source: imgur.com" /></a>

### Conclusion

Some steps may have been skipped to shorten the length of procedure section for simplicity. However, I hope this project was clear and concise for your review. Please feel free to contact me for feedback, suggestions, or comments.


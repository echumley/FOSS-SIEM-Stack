# Open-Source SIEM Deployment

This project allows for the quick deployment of a full-service log aggregation, SIEM, and SOAR stack using only free, open-source software. It utilizes Ansible to configure the hosts and Docker Compose to deploy the services.

## To-Do

- [x] Create Docker Compose file for logging stack
- [x] Create playbook for logging stack
- [x] Create playbook for Wazuh manager deployment
- [ ] Create playbook for logging & security agent installation (Fail2Ban, Crowdsec, Wazuh agent, Prometheus Node Exporter & Promtail Agent) for various OSs
- [ ] Create playbook for Zeek deployment
- [ ] Create playbook for TheHive & Cortex installation
- [ ] Create automatic integration script to connect all the necessary services
- [ ] Ensure Docker compatibility with internal logging & container security

## Log Pipeline

In order to quickly ingest logs, I'm using two log ingestion and aggregation pipelines: General and Security.

![alt text](https://github.com/echumley/FOSS-SIEM-Stack/blob/99313b9ac5624133c00f900c6e75f3850bf9ce1c/Security-Stack.drawio.png)

### General Log Pipeline

* **Prometheus Node Exporter:** Collects hardware and OS metrics from Linux machines, providing detailed system-level insights for monitoring.
* **Promtail:** Gathers and ships logs from various sources to Loki, facilitating structured log collection and forwarding.
* **Loki:** Provides a scalable, cost-efficient log storage and querying system, ideal for monitoring, troubleshooting, and correlating logs with metrics.
* **Grafana:** A powerful open-source platform for visualizing and analyzing metrics and logs, enabling the creation of rich, interactive dashboards.

### Security Log Pipeline

* **Crowdsec:** A collaborative, open-source intrusion prevention system that analyzes logs for malicious activity and shares threat intelligence to improve security.
* **Fail2Ban:** Monitors log files for suspicious activity, such as failed login attempts, and automatically bans IP addresses to prevent brute-force attacks.
* **Wazuh Agent:** The Wazuh agent installs on the host and forwards security logs and health information to the Wazuh Manager
* **Zeek:** A network security monitor that analyzes traffic in real time, producing detailed logs for detecting network anomalies and intrusions.
* **Wazuh Manager:** Focuses on security-related logs and threat detection, enhancing the system’s security posture.
* **TheHive:** An open-source Security Incident Response Platform (SIRP) that helps organize and manage security incidents, facilitating collaboration and case management among analysts.
* **Cortex:** Analysis and enrichment engine that automates the querying of threat intelligence sources and forensic tools to provide contextual data for security incidents.

## SIEM & SOAR

The SIEM and SOAR of choice for this project are Wazuh and TheHive due to their open-source model, continued support from developers and the community, rich feature-sets, and interoperability with other security-related services. Wazuh aggregates and correlates security logs from Fail2Ban, Crowdsec, and the Wazuh agent installed on the hosts. After logs are aggregated and correlated, Wazuh will alert an analyst a threat or attack is detected. Wazuh has some built-in features for automatically remediating basic security issues, but TheHive will do more in-depth remediation when a complex interference is required.

Wazuh, TheHive, and Cortex work together to create a comprehensive security operations workflow, combining threat detection, analysis, and incident response. Here’s how they integrate and complement each other:

### Security Stack Workflow

**1. Threat Detection (Wazuh)**

* **Log Collection:** Wazuh collects and analyzes security logs, system data, and threat intelligence from various sources (e.g., endpoints, servers, firewalls).
* **Threat Analysis:** Wazuh applies rules and machine learning to detect anomalies, malware, intrusion attempts, or policy violations.
* **Alert Generation:** When Wazuh detects suspicious activity or a security event, it generates an alert. These alerts contain detailed information, such as affected hosts, severity, and relevant log data.

**2. Alert Ingestion (TheHive + Cortext Analyzers)**

* **Alert Transfer:** Wazuh can be configured to send alerts to TheHive using its REST API. This can be automated through Wazuh’s integration or a connector.
* **Alert Processing:** TheHive ingests these alerts and converts them into cases or observables. Each case represents a potential security incident that needs investigation.

**3. Incident Response (TheHive)**

* **Case Management:** 
Security analysts use TheHive to manage and investigate the cases created from Wazuh alerts. They can:
* Assign cases to team members.
* Add notes, collaborate, and track progress.
* **Observable Analysis:** Each case may contain observables (e.g., IP addresses, hashes, domains) that can be enriched or analyzed with integrated tools like Cortex.
* **Automated Workflows:** TheHive supports creating playbooks and response workflows to streamline investigations and responses.

**4. Integration Workflow Example:**

1. **Wazuh Detection:** Wazuh detects a brute-force attack on an SSH service and generates an alert.
2. **Alert Sent to TheHive:** The alert is forwarded to TheHive via the Wazuh integration.
3. **Case Creation:** TheHive creates a case based on the alert and includes relevant observables (e.g., attacker IP, timestamp).
4. **Investigation in TheHive:** Analysts investigate the case, enrich observables using Cortex analyzers, and take actions like blocking the IP.
4. **Resolution:** TheHive updates the case status and logs the remediation steps taken.

### Benefits of Integrating Wazuh & TheHive

* **Centralized Incident Management:** TheHive provides a unified platform to manage, track, and document incidents detected by Wazuh.
* **Improved Collaboration:** Security teams can collaborate effectively on investigations with TheHive’s case management system.
* **Automated Threat Response:** The combination of Wazuh’s detection capabilities and TheHive’s incident response workflows helps automate and accelerate the response process.
* **Enhanced Threat Intelligence:** Integrating Cortex with TheHive allows enrichment of Wazuh alerts, providing deeper insights into threats.

## How to Use

### Step 1: The Logging Stack - COMPLETED, BUT NEEDS TUNING AND VALIDATION

1. Install Ansible & dependencies ([text](https://documentation.wazuh.com/current/deployment-options/deploying-with-ansible/guide/install-ansible.html))
2. Deploy a Debian or Ubuntu VM
3. Ensure you can reach the VM using Ansible
    ```
    ansible all -i inventory.ini -m ping
    ```
4. Deploy the SIEM logging stack using the `loggin-stack-setup.yml`
    ```
    ansible-playbook logging-stack-setup.yml -i inventory.ini --ask-become
    ```
5. 
ADD THE STEPS TO VERIFY THE STACK IS WORKING AS INTENDED

### Step 2: Deploy the Wazuh Server

1. Deploy another VM
2. Ensure the VM has a static IPv4 address
3. Deploy the Wazuh single node Ansible playbook
    ```
    ansible-playbook wazuh-setup.yml -i inventory.ini --ask-become
    ```
4. Look at the console output for the admin login information
5. Verify you can access the Wazuh manager by going to `https://<ip-address>`
6. Login using admin login information
7. To configure the Wazuh server further, modify the `/var/ossec/etc/ossec.conf`

### Step 3: Deploy the Security Tools on a Host - NOT COMPLETED

1. Deploy another VM (or use one you already have running, but I recommend deploying it on a test VM first)
2. Change the Wazuh server IPv4 address in `vm-setup.yml`
3. Deploy the `deploy-host-tools.yml`
    ```
    ansible-playbook deploy-agent-tools.yml -i inventory.ini --ask-become
    ```
4. 
ADD THE STEPS TO VERIFY THE AGENTS ARE WORKING AND FORWARDING AS INTENDED

### Step 4: Deploy Zeek - NOT COMPLETED

1. Deploy another VM (or use on the same VM, but might require some change in the Ansible Playbook)
2. Deploy `zeek-setup.yml`
    ```
    ansible-playbook zeek-setup.yml -i inventory.ini --ask-become
    ```
3. Verify zeek-compose.yml has correctly deployed
    ```
    sudo docker ps -a
    ```
4. Log into Zeek's webUI and configure

### Step 5: Deploy TheHive - NOT COMPLETED

1. Deploy another VM (or use on the same VM, but might require some change in the Ansible Playbook)
2. Deploy `thehive-setup.yml`
    ```
    ansible-playbook thehive-setup.yml -i inventory.ini --ask-become
    ```
3. Verify thehive-compose.yml has correctly deployed
    ```
    sudo docker ps -a
    ```
4. Log into TheHive's webUI at `https://ip/thehive`
5. Use TheHive's official [First Start Guide](https://docs.strangebee.com/thehive/administration/first-start/) to input a free community license and add an organization.

### Step 6: Deploy Cortex - NOT COMPLETED

1. Deploy another VM (or use on the same VM, but might require some change in the Ansible Playbook)
2. Deploy `cortex-setup.yml`
    ```
    ansible-playbook cortex-setup.yml -i inventory.ini --ask-become
    ```
3. Verify cortex-compose.yml has correctly deployed
    ```
    sudo docker ps -a
    ```
4. Log into Cortex's webUI and configure

## Useful Guides

https://documentation.wazuh.com/current/deployment-options/deploying-with-ansible/guide/install-ansible.html \
https://docs.strangebee.com/thehive/administration/first-start/
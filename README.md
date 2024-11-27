# Open-Source SIEM Deployment

This project allows for the quick deployment of a full-service log aggregation, SIEM, and SOAR stack using only free, open-source software. It utilizes Ansible to configure the hosts and Docker Compose to deploy the services.

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

### Log Workflow

**1. Log Collection and Forwarding:**

* Use Promtail, Prometheus Node Exporter, and the Wazuh agent to collect logs from Linux/Windows servers, routers, firewalls, and applications.
* Filter logs based on type (e.g., application logs go to Loki; security-related logs go to Wazuh).
* Forward logs in parallel to both Loki and Wazuh.

**2. Log Storage and Monitoring:**

* Store application and infrastructure logs in Loki for efficient querying, storage, and visualization.
* Use Grafana to create dashboards for application performance monitoring, incident analysis, and system health checks.

**3. Security Analysis:**

* Send security-related logs (e.g., auth logs, firewall logs) to Wazuh for advanced analysis.
* Detect and respond to anomalies, unauthorized access attempts, and compliance issues.

# Open-Source SIEM Deployment

This project allows for the quick deployment of a full-service log aggregration, SIEM, and SOAR stack using only free, open-source software. It utilizes Ansible to configure the hosts and Docker Compose to deploy the services.

## Log Pipeline

In order to quickly ingest logs, I'm using a 3-tiered log ingestion and aggregation pipeline.

* **rsyslog:** Handles log collection, forwarding, and preprocessing at the system level. It acts as the foundation, efficiently gathering logs from multiple sources and forwarding them to Loki and Wazuh.
* **Loki:** Provides a scalable, cost-efficient log storage and querying system, ideal for monitoring, troubleshooting, and correlating logs with metrics.
* **Wazuh:** Focuses on security-related logs and threat detection, enhancing the system’s security posture.

```
+-------------------+
|      Rsyslog      | <---- Collects logs from various sources
+-------------------+
        |
        v
+-------------------+      +-------------------+
|      Loki         | ---> |     Grafana       |
|  (Log Aggregator) |      |  (Visualization)  |
+-------------------+      +-------------------+
        |
        v
+-------------------+      +-------------------+      +---------------------+
|      Wazuh        | ---> |     Shuffle       | ---> |    Prometheus       |
| (Security Events) |      |   (Automation)    |      |  (Metrics & Health) |
+-------------------+      +-------------------+      +---------------------+
        |
        v
+-------------------+
|       Zeek        | ---> (Traffic Logs to Loki or Wazuh)
| (Network Traffic) |
+-------------------+
        |
        v
+-------------------+      +-------------------+
|   CrowdSec        | ---> |     Fail2Ban      | ---> (Threat Intelligence, IP Blocking)
| (Threat Analysis) |      |    (Brute-Force   |
+-------------------+      |     Protection)   |
                           +-------------------+
```

### Log Workflow

**1. Log Collection and Forwarding:**

* Use rsyslog to collect logs from Linux/Windows servers, routers, firewalls, and applications.
* Filter logs based on type (e.g., application logs go to Loki; security-related logs go to Wazuh).
* Forward logs in parallel to both Loki and Wazuh.

**2. Log Storage and Monitoring:**

* Store application and infrastructure logs in Loki for efficient querying, storage, and visualization.
* Use Grafana to create dashboards for application performance monitoring, incident analysis, and system health checks.

**3. Security Analysis:**

* Send security-related logs (e.g., auth logs, firewall logs) to Wazuh for advanced analysis.
* Detect and respond to anomalies, unauthorized access attempts, and compliance issues.
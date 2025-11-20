# Blue Team / Purple Team Homelab – Full-Stack Detection & Response Environment (2025 Edition)

### Document Purpose

This living document serves as the complete step-by-step build guide, configuration reference, and personal knowledge base for my advanced cybersecurity home lab focused on defensive security (Blue Team) and incident response (IR).

### Primary Learning Objectives

By completing this lab, I will demonstrate real-world proficiency in the following areas:

1. **Enterprise-grade logging & visibility**
– Collect, normalize, and search logs from Windows and Linux at scale using Splunk Enterprise
– Achieve near-complete endpoint and server visibility (process creation, network connections, file changes, authentication, etc.)
2. **Modern endpoint detection & response (EDR)**
– Deploy and manage an open-source EDR (OpenEDR) in a production-like environment
– Understand EDR telemetry gaps and fill them with Sysmon and custom configs
3. **Server hardening & deliberate attack surface creation**
– Secure an Apache web server on Ubuntu following CIS benchmarks
– Intentionally leave realistic entry points (misconfigurations, vulnerable apps, weak credentials) to practice detection and response
4. **Automated incident response orchestration (SOAR)**
– Build, test, and execute playbooks in Splunk SOAR
– Automate containment, enrichment, and notification workflows
5. **Professional incident case management**
– Run a full-featured, open-source incident response platform (DFIR-IRIS)
– Practice IOC tracking, task assignment, evidence vault management, timelines, and reporting—exactly like a real SOC or DFIR team
6. **End-to-end incident simulation**
– Perform controlled attacks from Kali → detect in Splunk → contain with SOAR → investigate and document in IRIS
– Close the loop from alert → detection → response → documentation

### 

Let’s go

[Master Stack Creation Document](Blue%20Team%20Purple%20Team%20Homelab%20%E2%80%93%20Full-Stack%20Detecti/Master%20Stack%20Creation%20Document%202aff484cbe9d808e830bf9557ef51758.md)

Setting up the Infra

[Infra Setup](Blue%20Team%20Purple%20Team%20Homelab%20%E2%80%93%20Full-Stack%20Detecti/Infra%20Setup%202aff484cbe9d80629e08c57e0c406a2c.md)
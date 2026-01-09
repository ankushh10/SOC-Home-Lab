☁️ AWS Cloud Security & Threat Hunting
CloudTrail → Wazuh SOC Integration
📌 Project Overview

This project demonstrates a Hybrid Cloud SOC (Security Operations Center) implementation by integrating AWS CloudTrail with Wazuh.

The goal is to detect and respond to:

Identity-based attacks

Unauthorized cloud resource usage

Defense evasion and logging tampering

The lab simulates real-world adversary behavior and validates detections end-to-end.

🏗️ Architecture

Log Source: AWS CloudTrail (Management Events)

AWS Region: us-east-1 (N. Virginia)

Log Storage: Amazon S3 (JSON logs)

Log Ingestion: Wazuh Manager (Docker + AWS S3 module)

Visualization: Kibana / OpenSearch Dashboards

📁 Repository Structure
aws-wazuh-threat-hunting/
├── README.md
├── iam/
│   └── Read_WazuhS3Bucket.json
├── wazuh/
│   └── ossec.conf
└── screenshots/
    ├── dashboard.png
    └── console-login-failure.png

🛡️ Implementation Details
1️⃣ Least Privilege IAM Design

To follow security best practices, no admin credentials were used.

A dedicated IAM user named wazuh-log-collector was created with read-only access to the CloudTrail S3 bucket.

IAM Policy Location:
iam/Read_WazuhS3Bucket.json

Policy Purpose:

Allow listing the CloudTrail bucket

Allow reading log objects

Prevent write, delete, or privilege escalation actions

2️⃣ Wazuh Configuration (AWS S3 Module)

The Wazuh Manager runs in Docker and pulls CloudTrail logs directly from the S3 bucket.

Configuration File:
wazuh/ossec.conf

Relevant configuration snippet:

<wodle name="aws-s3">
  <disabled>no</disabled>
  <interval>10m</interval>
  <run_on_start>yes</run_on_start>
  <skip_on_error>yes</skip_on_error>

  <bucket type="cloudtrail">
    <name><YOUR_BUCKET_NAME></name>
    <regions>us-east-1</regions>
  </bucket>
</wodle>

⚔️ Attack Simulation & Detection

To validate the SOC pipeline, multiple real-world attack scenarios were simulated against the AWS environment.

🚨 Simulated Threat Scenarios
Attack Scenario	AWS Event Name	Wazuh Rule ID	Severity	Description
Identity Attack	ConsoleLogin (Failure)	80200	Medium	Multiple failed AWS Console login attempts
Data Exposure	PutBucketPublicAccessBlock	80202	High	Attempt to remove S3 Block Public Access
Shadow IT / Crypto Mining	RunInstances	80301	Medium	Unauthorized EC2 instance launch
Defense Evasion	StopLogging	80202	Critical	Attempt to disable CloudTrail
📸 Evidence of Detection
📊 Threat Hunting Dashboard

Custom dashboards were created to visualize AWS alert categories and severity.

Screenshot location:

screenshots/dashboard.png

🔍 Log Analysis: Console Login Failure

Drill-down analysis of failed AWS Console login attempts detected by Wazuh.

Screenshot location:

screenshots/console-login-failure.png

🔧 Incident Response Playbook
🚨 Account Compromise — Rule ID: 80200

Lock the affected IAM user

Rotate access keys immediately

Force password reset

Enforce MFA

🖥️ Unauthorized Compute Usage — Rule ID: 80301

Terminate the unauthorized EC2 instance

Review Security Group rules for exposure

Audit CloudTrail for additional launches

🛑 Logging Tampering / Defense Evasion — Rule ID: 80202

Re-enable CloudTrail immediately

Audit the logging gap window

Investigate lateral movement or data exfiltration

🎯 Key Takeaways

End-to-end AWS CloudTrail → Wazuh detection pipeline

Least-privilege IAM implementation

Realistic adversary simulation

SOC-ready incident response workflows

🚀 Future Enhancements

MITRE ATT&CK technique mapping

Automated remediation using Lambda

GuardDuty + Wazuh correlation

Detection-as-code versioning

# ☁️ AWS Cloud Security & Threat Hunting (Wazuh Integration)

## 📌 Project Overview
This project demonstrates the implementation of a **Hybrid Cloud SOC** (Security Operations Center) by integrating **AWS CloudTrail** with **Wazuh**. The goal was to monitor cloud infrastructure for identity attacks, unauthorized resource usage, and defense evasion attempts in real-time.

### 🏗️ Architecture
* **Log Source:** AWS CloudTrail (Management Events) monitoring `us-east-1` (N. Virginia).
* **Storage:** AWS S3 Bucket (JSON formatted logs).
* **Ingestion:** Wazuh Manager (Python Boto3 Module/Docker).
* **Visualization:** Custom Kibana/OpenSearch Dashboards.

---

## 🛡️ Implementation Details

### 1. Least Privilege IAM Policy
To ensure security best practices, I avoided using Admin keys. Instead, I created a custom IAM user (`wazuh-log-collector`) with a restricted policy that only allows reading from the specific logging bucket.

**Policy: `Read_WazuhS3Bucket`**
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "WazuhBucketListing",
            "Effect": "Allow",
            "Action": "s3:ListBucket",
            "Resource": "arn:aws:s3:::<YOUR_BUCKET_NAME>"
        },
        {
            "Sid": "WazuhLogReading",
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::<YOUR_BUCKET_NAME>/*"
        }
    ]
}
```

### 2. Wazuh Configuration
Configured the Wazuh Manager (running in Docker) to ingest logs via the AWS S3 module.

**File: `ossec.conf`**
```xml
<wodle name="aws-s3">
  <disabled>no</disabled>
  <interval>10m</interval>
  <run_on_start>yes</run_on_start>
  <skip_on_error>yes</skip_on_error>
  <bucket type="cloudtrail">
    <name><YOUR_BUCKET_NAME></name>
    <access_key><HIDDEN></access_key>
    <secret_key><HIDDEN></secret_key>
    <regions>us-east-1</regions>
  </bucket>
</wodle>
```

---

## ⚔️ Attack Simulation & Detection
To validate the detection pipeline, I executed four specific "Real World" attack scenarios against the AWS environment.

| Attack Scenario | AWS Event Name | Wazuh Rule ID | Severity | Description |
| :--- | :--- | :--- | :--- | :--- |
| **Identity Attack** | `ConsoleLogin` (Failure) | **80200** | Medium | Simulated a Brute Force attack by attempting to login with incorrect credentials multiple times. |
| **Data Exposure** | `PutBucketPublicAccessBlock` | **80202** | High | Detected an attempt to remove "Block Public Access" settings from a sensitive S3 bucket. |
| **Shadow IT / Mining** | `RunInstances` | **80301** | Medium | Detected the unauthorized launch of an EC2 instance (common tactic for crypto-jacking). |
| **Defense Evasion** | `StopLogging` | **80202** | Critical | Detected an adversary attempting to blind the SOC by turning off CloudTrail logging. |

---

## 📸 Evidence of Detection

### Dashboard View: Threat Hunting

![AWS Dashboard Screenshot](./images/aws-dashboard.png)


### Log Analysis: Console Login Failure


![Console Login Failure](./images/console-failure.png)

---

## 🔧 Incident Response Playbook
If these alerts trigger in a production environment, the following remediation steps are taken:

1.  **For Account Compromise (Rule 80200):**
    * Lock the IAM User account immediately.
    * Rotate Access Keys and force a password reset.
    * Enable MFA if not already enforced.

2.  **For Unauthorized Compute (Rule 80301):**
    * Terminate the unauthorized EC2 instance via CLI.
    * Review Security Group rules for any backend doors created.

3.  **For Logging Tampering (Rule 80202):**
    * Re-enable CloudTrail immediately.
    * Audit the specific time window (from Stop to Start) for lateral movement or data exfiltration.

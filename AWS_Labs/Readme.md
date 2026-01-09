# ☁️ AWS Cloud Security & Threat Hunting (Wazuh Integration)

## 📌 Project Overview
This project demonstrates the implementation of a **Hybrid Cloud SOC (Security Operations Center)** by integrating **AWS CloudTrail** with **Wazuh**.  
The objective is to monitor AWS cloud infrastructure for:

- Identity-based attacks  
- Unauthorized resource usage  
- Defense evasion attempts  

All detections are performed in near real-time using centralized log ingestion and visualization.

---

## 🏗️ Architecture

- **Log Source:** AWS CloudTrail (Management Events)
- **Region:** `us-east-1` (N. Virginia)
- **Storage:** AWS S3 Bucket (JSON formatted logs)
- **Ingestion:** Wazuh Manager (Docker + AWS S3 module)
- **Visualization:** Kibana / OpenSearch Dashboards

---

## 🛡️ Implementation Details

### 1️⃣ Least Privilege IAM Policy

To follow security best practices, **no admin credentials** were used.

A dedicated IAM user named **`wazuh-log-collector`** was created with **read-only access** to the CloudTrail S3 bucket.

#### IAM Policy: `Read_WazuhS3Bucket`

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

2️⃣ Wazuh Configuration (AWS S3 Module)
The Wazuh Manager, running in Docker, was configured to ingest CloudTrail logs directly from the S3 bucket.
File: ossec.conf
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


⚔️ Attack Simulation & Detection
To validate the detection pipeline, real-world attack scenarios were simulated against the AWS environment.
🚨 Simulated Attack Scenarios
Attack ScenarioAWS Event NameWazuh Rule IDSeverityDescriptionIdentity AttackConsoleLogin (Failure)80200MediumSimulated brute-force login attempts with invalid credentialsData ExposurePutBucketPublicAccessBlock80202HighAttempted removal of S3 Block Public Access settingsShadow IT / Crypto MiningRunInstances80301MediumUnauthorized EC2 instance launchDefense EvasionStopLogging80202CriticalAttempt to disable CloudTrail logging

📸 Evidence of Detection
📊 Dashboard View — Threat Hunting
Custom visualizations were created to track AWS-specific alert groups.

📌 Insert screenshot of your pie chart / dashboard here


🔍 Log Analysis — Console Login Failure
Drill-down view of failed AWS Console login attempts detected by Wazuh.

📌 Insert screenshot of the specific log entry here


🔧 Incident Response Playbook
🚨 Account Compromise (Rule ID: 80200)


Immediately lock the affected IAM user


Rotate all access keys


Force password reset


Enable MFA if not already enforced



🖥️ Unauthorized Compute Usage (Rule ID: 80301)


Terminate the unauthorized EC2 instance via AWS CLI or Console


Review Security Group rules for backdoors or exposed services


Check CloudTrail for additional instance launches



🛑 Logging Tampering / Defense Evasion (Rule ID: 80202)


Re-enable CloudTrail immediately


Audit the time window between StopLogging and StartLogging


Investigate potential lateral movement or data exfiltration



✅ Key Takeaways


Demonstrates real-world AWS threat detection using CloudTrail


Implements least-privilege IAM design


Validates SOC workflows with simulated adversary behavior


Provides actionable incident response steps




If you want next:
- 🔥 **Add MITRE ATT&CK mappings**
- 📁 **Split into `/architecture`, `/detections`, `/playbooks`**
- 🧠 **Make it resume-ready with “What this proves” bullets**

Just say the word.

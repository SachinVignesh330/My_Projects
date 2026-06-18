**Real-Time AWS Security Monitoring Pipeline**

This project is an automated cloud security watchdog. It intercepts IAM role modifications in real time, evaluates permissions against known privilege escalation signatures, archives audit logs to S3, and fires instant alerts via SNS to ensure immediate visibility into environment changes.

---

**Available Components**

* **Event Routing:** AWS EventBridge global cross-region routing rule configurations.
* **Analysis Engine:** AWS Lambda function runtime setup using Python 3.12 and the Boto3 SDK.
* **Storage Layer:** Amazon S3 bucket integration for persistent JSON audit log storage.
* **Alerting System:** Amazon SNS messaging configuration with direct email integration.

---

**Prerequisites**

* **AWS Account:** Active AWS account with permissions to deploy Lambda, EventBridge, S3, and SNS resources.
* **IAM Permissions:** Adequate administrative permissions to read target IAM policies and execution metadata.
* **Python Engine:** Python 3.8+ compatibility if managing or running the deployment scripts locally.
* **Verified Endpoints:** A confirmed email address subscription mapped to your target Amazon SNS Topic.

> **Note:** This implementation primarily involves automating cloud infrastructure security auditing. The resources focus on processing real-time event patterns rather than hosting persistent virtual machinery.

---

**AWS Infrastructure Configurations**

**1. Amazon S3 (Compliance Storage Bucket)**
* **Bucket Policy:** Configured to strictly allow write access (s3:PutObject) from the Lambda execution role while enforcing encryption-in-transit via HTTPS.
* **Directory Structure:** Organised to systematically generate chronological pathways using the pattern: findings/YYYY-MM-DD/.
* **Lifecycle Rule:** Enabled a 90-day transition to Glacier Flexible Retrieval to optimize corporate storage compliance costs.

**2. AWS Lambda (The Python Analysis Engine)**
* **Runtime Environment:** Configured natively on Python.
* **Timeout Settings:** Adjusted the execution limit to 1 minute (default is 3 seconds) to ensure plenty of overhead for processing deep nested IAM loops.
* **Environment Configuration:** Securely injected the targets using environment variables (AUDIT_BUCKET and SNS_TOPIC_ARN), completely avoiding hardcoded strings.

**3. Amazon SNS (The Notification Engine)**
* **Topic Type:** Standard Topic configuration allowing highly concurrent, real-time fan-out notifications.
* **Access Policy:** Configured with an explicit resource policy granting sns:Publish access exclusively to your specific Lambda function ARN.
* **Subscription:** Configured via Email protocol. Ensure the status shows Confirmed in your AWS console, or alerts will be silently dropped.

**4. Amazon EventBridge (The Event Bus Routing)**
* **Global Rule (us-east-1):** Intercepts global IAM API patterns from CloudTrail using a designated JSON event pattern targeting CreateRole, AttachRolePolicy, and PutRolePolicy events.
* **Target Mapping:** Set up to forward matching events directly across regions to my localized EventBridge bus in my home region.

---

**Engineering Challenges & Troubleshooting**

**1. Cross-Region Event Loss**
* **Problem:** During initial testing, IAM modification events were completely missing from the pipeline. Because AWS IAM is a global service, its API logs default to the N. Virginia (us-east-1) data center. Since my core Lambda function was running locally in Mumbai (ap-south-1), the events weren't making it across the pond, resulting in silent drops.
* **Solution:** I set up a cross-region event-routing framework using Amazon EventBridge and built a routing rule in the us-east-1 default bus to catch the relevant IAM configuration patterns and forward them directly to the localized EventBridge bus in ap-south-1. This fixed the visibility issue and restored data flow across regions.

**2. Runtime Script Interruptions and String Incompatibilities**
* **Problem:** The system was throwing errors and crashing right when trying to trigger email alerts. When I checked CloudWatch, I found syntax mismatches in the resource naming schemas across my environment—specifically, minor typo discrepancies using underscores instead of dashes in the Amazon Resource Names (ARNs), combined with an incorrect storage pointer for the S3 bucket.
* **Solution:** I decoupled the configuration settings from the code and  stripped out all hardcoded resource strings and shifted them into system-level Lambda Environment Variables (specifically AUDIT_BUCKET and SNS_TOPIC_ARN). This fixed the bugs, made the codebase modular, and ensured smooth runtime processing.

**3. Redundant Overlapping Alarms Under Full-Admin Contexts**
* **Problem:** When I deployed a highly permissive profile (like attaching the standard AdministratorAccess policy) to a test role, the engine went haywire and triggered every single security violation signature simultaneously. At first glance, it looked like a logic loop flaw or a massive wave of false positives in my conditional blocks.
* **Solution:** Did a deep-dive trace of the backend evaluation script's logic. It turns out the engine was actually working exactly as designed. Because an Administrator policy uses a wildcard character ("*") to grant all permissions, that test role technically satisfied every privilege escalation vector at the exact same time. The cascading alerts were completely valid, proving that the monitoring logic takes a strict, zero-trust security stance.

---


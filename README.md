# cloudformation-detect-and-notify-team-after-aws-keys-are-rotated
Stack description
Account-agnostic CloudFormation stack to detect AWS Secrets Manager rotation events and notify teams via SNS. Designed for reuse across multiple AWS accounts and regulated (GovCloud-style) environments.

## What This Stack Does

- Runs on a fixed **hourly schedule**
- Scans **Secrets Manager metadata only**
- Detects **any secret change or rotation**
- Sends **one aggregated email notification** when changes occur
- Remains **silent** when no changes are detected
- Uses **least-privilege IAM permissions**
- Does **not** read secret values or key material

---

## High-Level Architecture

EventBridge Scheduler (hourly)
        |
        v
AWS Lambda (inline code)
        |
        v
AWS Secrets Manager (metadata only)
        |
        v
Amazon SNS Topic
        |
        v
Email subscriptions


All components are created and managed by a single CloudFormation stack.

---

## Key Design Principles

- **Account-agnostic**  
  No AWS account IDs are passed as parameters. The stack auto-detects the account and region using CloudFormation pseudo parameters.

- **System-agnostic**  
  A system label (e.g. PROJECT_NAME_HERE ) is provided as a parameter for alert clarity, not logic branching.

- **Environment-aware, not environment-dependent**  
  Environment names (dev, int, staging, prod, etc.) are labels only and do not alter detection logic.

- **Metadata-only access**  
  The Lambda function reads timestamps and ARNs only. No secret values are accessed.

- **Low-noise operations**  
  Notifications are sent only when a change is detected.

---

## CloudFormation Parameters

The stack is configured entirely through parameters at deploy time.

### Required Parameters

- **SystemName**  
  Logical platform app or system name 

- **Environment**  
  Logical environment label  
  Allowed values:
sandbox, dev, int, staging, uat, prod


- **AlertEmail**  
Primary email address for SNS notifications

### Optional Parameters

- **AlertEmail2Optional**  
Secondary email address for SNS notifications  
If left blank, no second subscription is created.

---

## Email Notification Format

### Subject

Secret Rotation Detected – <system>-<environment>


Example:
Secret Rotation Detected – cdw-dev


### Body (example)

System: INSERT_SYSTEM_NAME_HERE
Environment: dev
AWS Account ID: 123456789012
AWS Region: us-gov-west-1
Detection Window: last 1 hour

Secrets Changed (2):

Secret Name: example-db-credentials
ARN: arn:aws:secretsmanager:...
Last Changed (ET): 2026-02-05 14:12:03 EST


Only metadata is included.

---

## Stack Components

### 1. EventBridge Scheduler
- Fixed rate: `1 hour`
- Time-deterministic
- Fully managed via CloudFormation

### 2. AWS Lambda (inline)
- Embedded directly in the template (`ZipFile`)
- Environment-driven configuration
- No hardcoded account, system, or org identifiers

### 3. IAM Role
- `secretsmanager:ListSecrets`
- `secretsmanager:DescribeSecret`
- `sns:Publish`
- CloudWatch Logs permissions
- No permissions to read secret values

### 4. SNS Topic
- One topic per system/environment
- Email protocol subscriptions
- Supports fan-out via additional subscribers

---

## Deployment Model

The same template can be deployed:
- Across multiple AWS accounts
- Across different platforms (MiHub, CDW, etc.)
- Across multiple environments

Only parameter values change — the template itself does not.

---

## Customization Options

Common adjustments:
- Change schedule frequency
- Add high-priority secret name patterns
- Integrate with ticketing or incident systems
- Automate SNS subscriptions via CLI or scripts

All customizations can be made without modifying the core detection logic.

---

## Safety & Reuse

This stack:
- Contains no credentials
- Contains no internal business logic
- Uses AWS-managed services only
- Is safe to share as a reference architecture or utility pattern

It is intended as **infrastructure tooling**, not application code.

---

## Summary

This CloudFormation stack provides a simple, reliable pattern for:
- Detecting secret rotation events
- Notifying teams with clear context
- Operating consistently across accounts and environments
- Reducing manual monitoring and configuration drift
git push -uf origin main
```

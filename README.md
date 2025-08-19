# AWS-multi-account

In my last project, we migrated from a single-account setup to an AWS multi-account model using AWS Organizations. We set up separate accounts for Prod, Dev, and Security, enforced compliance using Service Control Policies, and centralized logging in a Log Archive Account. This reduced our security risk (by isolating blast radius) and improved cost visibility by 30%. Developers got their own sandbox accounts with guardrails, while production was tightly controlled.

---

# 📘 Managing Multiple AWS Accounts – How & Why

## 1. **Why Multiple AWS Accounts?**

Using a single AWS account for everything can quickly lead to problems. AWS recommends a **multi-account strategy** for better **security, cost control, compliance, and scalability**.

### ✅ Benefits:

* **Security Isolation** – Blast radius is limited (e.g., production vs dev/test).
* **Cost Management** – Easier to track spending by environment/team/project.
* **Compliance & Governance** – Helps align with regulatory needs (HIPAA, PCI, GDPR).
* **Service Quotas** – Each account has its own limits (helps avoid bottlenecks).
* **Delegated Ownership** – Teams can manage their own accounts with guardrails.
* **Separation of Concerns** – Dev, QA, Prod, and sandbox accounts can be independent.

📌 Example: Instead of mixing test and prod databases in the same account, you create separate accounts. Even if dev gets compromised, prod remains safe.

---

## 2. **Best Practices for Multi-Account Management**

### 🏛️ Use **AWS Organizations**

* **Centralized management service** for accounts.
* Enables:

  * **Consolidated Billing** – Single invoice, cost visibility by account.
  * **Service Control Policies (SCPs)** – Set guardrails (e.g., deny disabling CloudTrail).
  * **Organizational Units (OUs)** – Group accounts by function/env (Prod, Dev, Security).

**OU Example Structure:**

```
Root
├── Security OU
│   ├── Log Archive Account
│   └── Security Tools Account
├── Infrastructure OU
│   ├── Networking Account (VPCs, Transit Gateway)
│   └── Shared Services Account (AD, CI/CD tools)
├── Sandbox OU
│   ├── Developer 1 Account
│   └── Developer 2 Account
└── Workloads OU
    ├── Dev Account
    ├── QA Account
    └── Prod Account
```

---

### 🔑 Centralized Identity & Access

* Use **AWS IAM Identity Center (SSO)** or external IdP (Okta, Azure AD).
* Users authenticate once, then assume roles in target accounts.
* Enforce **MFA everywhere**.

---

### 📊 Centralized Logging & Monitoring

* Send all logs (CloudTrail, Config, GuardDuty) to a **Log Archive Account**.
* Use **AWS Security Hub, GuardDuty, and Detective** from a **Security Tools Account**.
* Use **CloudWatch cross-account dashboards** or external tools (Datadog, Splunk).

---

### 💰 Cost Management

* Enable **AWS Cost Explorer** and **Budgets**.
* Tagging strategy across accounts (`Environment`, `Application`, `Owner`).
* Use **Consolidated Billing** for volume discounts.

---

### 🔐 Security Best Practices

* Enforce least privilege IAM roles.
* Use **Service Control Policies (SCPs)**:

  * Deny root user usage.
  * Enforce specific regions (e.g., only `us-east-1`).
  * Prevent disabling security tools.
* Isolate Prod from Dev/Test accounts.

---

### 🔄 Networking Between Accounts

* Centralize networking in a **Networking Account** (VPCs, Transit Gateway).
* Use **AWS PrivateLink or VPC Peering** for secure inter-account communication.
* Avoid full mesh networking – centralize for simplicity.

---

### ⚙️ Automation & IaC

* Use **Terraform, AWS CDK, or CloudFormation** for account provisioning.
* Define baseline guardrails: IAM roles, CloudTrail, Config rules, SCPs.
* Use **Control Tower** (if starting fresh) to bootstrap a secure multi-account environment.

---

## 3. **How to Manage Multiple AWS Accounts (Step by Step)**

1. **Set up AWS Organizations** (one management/root account).
2. **Define Organizational Units (OUs)** → Prod, Dev, Security, Sandbox.
3. **Enable AWS Control Tower** (optional, but simplifies governance).
4. **Centralize Identity** → IAM Identity Center or external IdP.
5. **Create Guardrails with SCPs** → enforce compliance.
6. **Set up Centralized Logging** (Log Archive Account).
7. **Configure Security Tools Account** → GuardDuty, Security Hub, Detective.
8. **Provision Workload Accounts** → Dev, QA, Prod via IaC.
9. **Establish Cross-Account Networking** → Transit Gateway or VPC Peering.
10. **Apply Cost Controls** → Budgets, tagging, consolidated billing.

---

## 4. **Tools for Multi-Account Management**

* **AWS Control Tower** → Automates account setup, governance.
* **AWS Organizations** → Multi-account structure & billing.
* **AWS IAM Identity Center** → Centralized authentication/SSO.
* **Terraform/CloudFormation/CDK** → Infrastructure as Code across accounts.
* **Third-Party Tools** → Cloud Custodian, HashiCorp Vault, Datadog, Splunk.

---

## 5. **Interview/Real-World Story Angle**

> “In my last project, we migrated from a single-account setup to an **AWS multi-account model** using **AWS Organizations**. We set up separate accounts for Prod, Dev, and Security, enforced compliance using **Service Control Policies**, and centralized logging in a **Log Archive Account**. This reduced our security risk (by isolating blast radius) and improved cost visibility by 30%. Developers got their own sandbox accounts with guardrails, while production was tightly controlled.”

---



# Managing Multiple AWS Accounts: Best Practices and Documentation

## Why Manage Multiple AWS Accounts?

Managing multiple AWS accounts is a common practice for organizations to enhance security, optimize costs, improve governance, and streamline operations. Below are the key reasons why multiple AWS accounts are used:

- **Security Isolation**: Separate accounts reduce the blast radius of security incidents. For example, a compromised account is limited to its resources, protecting other environments (e.g., production vs. development).
- **Cost Allocation and Tracking**: Different accounts can be tied to specific projects, departments, or teams, making it easier to track and allocate costs accurately using AWS Cost Explorer or billing tags.
- **Environment Separation**: Segregating development, staging, and production environments into different accounts ensures better resource isolation and minimizes accidental interference.
- **Compliance and Governance**: Different accounts can be configured to meet specific compliance requirements (e.g., HIPAA, GDPR) or organizational policies, with tailored IAM roles and permissions.
- **Scalability and Flexibility**: Multiple accounts allow teams to operate independently, enabling faster development cycles and easier management of large-scale deployments.
- **Quota and Limit Management**: AWS service quotas are account-specific. Multiple accounts prevent hitting limits for resources like EC2 instances or Lambda functions in critical environments.

## How to Manage Multiple AWS Accounts

AWS provides tools and best practices to manage multiple accounts efficiently. Below is a comprehensive guide to setting up and managing multiple AWS accounts.

### 1. Use AWS Organizations
AWS Organizations is a service that allows you to centrally manage and govern multiple AWS accounts. It simplifies account management, billing, and policy enforcement.

- **Key Features**:
  - **Consolidated Billing**: Aggregate billing across accounts under a single management (payer) account to simplify cost tracking and take advantage of volume discounts.
  - **Organizational Units (OUs)**: Group accounts into OUs based on function (e.g., Dev, Prod, Finance) for easier policy application.
  - **Service Control Policies (SCPs)**: Apply permissions boundaries to control what actions accounts or OUs can perform, enhancing security and compliance.
  - **Account Creation Automation**: Automate the creation of new AWS accounts using AWS Organizations APIs or AWS CLI.

- **Setup Steps**:
  1. Sign in to the AWS Management Console with the management account.
  2. Enable AWS Organizations by navigating to the AWS Organizations service and selecting "Create an organization."
  3. Create OUs to group accounts (e.g., `Dev`, `Prod`, `Shared-Services`).
  4. Invite existing AWS accounts or create new ones under the organization.
  5. Apply SCPs to OUs or individual accounts to enforce policies (e.g., restrict regions or services).
  6. Enable consolidated billing to centralize payment methods.

- **Best Practices**:
  - Use a dedicated management account with minimal resources to reduce risk.
  - Enable AWS CloudTrail across all accounts and centralize logs in a dedicated logging account.
  - Tag accounts and OUs for better cost allocation and tracking (e.g., `Department: Marketing`, `Environment: Prod`).

### 2. Implement Centralized Identity Management with AWS IAM Identity Center
AWS IAM Identity Center (successor to AWS Single Sign-On) allows centralized user access management across multiple AWS accounts.

- **Key Features**:
  - Single sign-on (SSO) for accessing multiple accounts with one set of credentials.
  - Integration with external identity providers (e.g., Active Directory, Okta, Azure AD).
  - Role-based access control (RBAC) for managing permissions across accounts.

- **Setup Steps**:
  1. Enable IAM Identity Center in the management account.
  2. Configure an identity source (e.g., IAM Identity Center’s identity store or an external IdP).
  3. Create permission sets defining roles (e.g., Admin, Developer, ReadOnly).
  4. Assign users or groups to accounts and permission sets.
  5. Enable SSO access for users via the AWS access portal.

- **Best Practices**:
  - Use least privilege principles when defining permission sets.
  - Regularly audit user access and remove unused accounts or permissions.
  - Integrate with multi-factor authentication (MFA) for enhanced security.

### 3. Centralize Logging and Monitoring
Centralized logging and monitoring ensure visibility into activities and resource usage across accounts.

- **Tools**:
  - **AWS CloudTrail**: Log API calls across all accounts and store logs in a centralized S3 bucket in a dedicated logging account.
  - **Amazon CloudWatch**: Aggregate metrics, logs, and alarms from all accounts into a central monitoring account.
  - **AWS Config**: Track resource configurations and compliance across accounts.

- **Setup Steps**:
  1. Create a dedicated logging account in AWS Organizations.
  2. Enable CloudTrail in all accounts and configure it to send logs to an S3 bucket in the logging account.
  3. Set up CloudWatch cross-account observability to aggregate metrics and logs.
  4. Use AWS Config to monitor compliance with organizational policies (e.g., encryption requirements).

- **Best Practices**:
  - Enable encryption for logs stored in S3.
  - Set up CloudWatch alarms for critical events (e.g., unauthorized API calls).
  - Use AWS Security Hub to aggregate security findings across accounts.

### 4. Automate Account Management with Infrastructure as Code (IaC)
Use IaC tools like AWS CloudFormation or Terraform to automate account setup, resource provisioning, and policy enforcement.

- **Key Tools**:
  - **AWS CloudFormation**: Create templates to provision resources consistently across accounts.
  - **AWS Control Tower**: Automate the setup of a secure, multi-account AWS environment with predefined guardrails.
  - **Terraform**: Manage multi-account infrastructure using a single configuration.

- **Setup Steps**:
  1. Deploy AWS Control Tower from the management account to set up a landing zone with preconfigured accounts, OUs, and guardrails.
  2. Use CloudFormation StackSets to deploy resources across multiple accounts and regions.
  3. For Terraform, use modules to define reusable configurations for accounts and resources.

- **Best Practices**:
  - Store IaC templates in a version-controlled repository (e.g., AWS CodeCommit, GitHub).
  - Use AWS Control Tower for a standardized multi-account setup.
  - Test IaC templates in a sandbox account before deploying to production.

### 5. Secure Cross-Account Access
Cross-account access is often required for shared resources (e.g., a central S3 bucket or VPC). Use IAM roles and policies to manage access securely.

- **Setup Steps**:
  1. Create an IAM role in the target account with the necessary permissions.
  2. Configure a trust relationship in the IAM role to allow access from the source account.
  3. Grant the source account’s users or services permission to assume the role using `sts:AssumeRole`.

- **Best Practices**:
  - Use the `aws:PrincipalOrgID` condition in trust policies to restrict role assumption to accounts within the organization.
  - Enable MFA for cross-account role assumption.
  - Regularly rotate credentials and review role permissions.

### 6. Cost Management Across Accounts
Managing costs across multiple accounts requires clear visibility and governance.

- **Tools**:
  - **AWS Cost Explorer**: Analyze costs by account, OU, or tag.
  - **AWS Budgets**: Set budget alerts for accounts or OUs.
  - **AWS Cost Allocation Tags**: Tag resources for detailed cost tracking.

- **Setup Steps**:
  1. Enable cost allocation tags in the management account.
  2. Apply consistent tagging policies across accounts (e.g., `Project`, `Environment`).
  3. Use Cost Explorer to create reports for cost analysis by account or OU.
  4. Set up AWS Budgets to monitor spending and receive alerts.

- **Best Practices**:
  - Enforce tagging policies using SCPs or AWS Config rules.
  - Review cost reports regularly to identify cost-saving opportunities (e.g., unused resources).
  - Use AWS Trusted Advisor to identify underutilized resources across accounts.

### 7. Backup and Disaster Recovery
Ensure data and resources are protected across all accounts.

- **Tools**:
  - **AWS Backup**: Centralize backup policies across accounts for services like EBS, RDS, and S3.
  - **AWS Disaster Recovery**: Use services like AWS Elastic Disaster Recovery for cross-account recovery.

- **Setup Steps**:
  1. Create a backup vault in a dedicated backup account.
  2. Define backup policies in AWS Backup and apply them to accounts or OUs.
  3. Test recovery procedures regularly to ensure data integrity.

- **Best Practices**:
  - Store backups in a separate account to protect against accidental deletion.
  - Enable cross-region replication for critical data.
  - Use versioning for S3 buckets to prevent data loss.

To format the provided AWS Organizations SCP (Service Control Policy) example for inclusion in well-structured notes, I'll ensure it’s clearly presented within a markdown document, maintaining the artifact structure and adhering to the requirement to wrap the SCP content in an `<xaiArtifact>` tag. The notes will include a brief explanation of the SCP’s purpose and context, formatted for clarity and readability.

<xaiArtifact artifact_id="947da377-7c64-43eb-a41b-85687af31f9b" artifact_version_id="59ae2ed8-eb40-4998-b0d6-bccc73a1c4d1" title="AWS SCP Example Notes.md" contentType="text/markdown">

# AWS Organizations SCP Example: Restricting Regions

## Overview
Service Control Policies (SCPs) in AWS Organizations allow you to set permission boundaries for accounts within your organization. The following example demonstrates an SCP that restricts AWS API actions to specific regions, ensuring that resources can only be created or managed in approved regions (e.g., `us-east-1` and `us-west-2`). This is useful for compliance, cost management, and operational consistency.

## SCP Example: Restrict Accounts to Specific Regions

The SCP below denies all actions in regions other than `us-east-1` and `us-west-2`. This policy can be applied to an Organizational Unit (OU) or individual accounts within AWS Organizations.
```
<xaiArtifact artifact_id="6387077b-5b4b-4323-bd92-3d105ab37134" artifact_version_id="a3f1226e-c4f5-4216-b4bc-4ffc87883635" title="restrict-region-scp.json" contentType="application/json">
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "*",
      "Resource": "*",
      "Condition": {
        "StringNotEquals": {
          "aws:RequestedRegion": [
            "us-east-1",
            "us-west-2"
          ]
        }
      }
    }
  ]
}
</xaiArtifact>

```

## Explanation of the SCP
- **Version**: Specifies the policy language version (`2012-10-17` is the current standard).
- **Effect: Deny**: Denies any action that doesn’t meet the condition.
- **Action: "*"**: Applies to all AWS actions.
- **Resource: "*"**: Applies to all resources.
- **Condition: StringNotEquals**: Ensures the policy only allows actions in the specified regions (`us-east-1`, `us-west-2`). Any action attempted in other regions is denied.

## Use Case
This SCP is ideal for:
- Ensuring compliance with regional data residency requirements.
- Reducing costs by limiting resource deployment to approved regions.
- Simplifying operations by standardizing on specific regions.

## Applying the SCP
1. Sign in to the AWS Management Console with the AWS Organizations management account.
2. Navigate to AWS Organizations > Policies > Service control policies.
3. Create a new SCP or edit an existing one.
4. Copy the JSON policy above into the policy editor.
5. Attach the SCP to the desired OU or account.

## Best Practices
- Test SCPs in a sandbox account before applying to production.
- Combine with other SCPs for granular control (e.g., allow specific services).
- Monitor denied actions using AWS CloudTrail to troubleshoot issues.

</xaiArtifact>
